






```
# ---------- Mongo client (lazy, cached) ----------
@lru_cache(maxsize=1)
def _get_mongo_client() -> MongoClient:
    """Create and cache a MongoClient instance per process."""
    try:
        client = MongoClient(MONGO_URI, serverSelectionTimeoutMS=2000)
        # trigger server selection to raise early if connection fails
        client.admin.command("ping")
        return client
    except Exception:
        # Log and re-raise so callers can handle it
        frappe.log_error(frappe.get_traceback(), "Mongo Client Init Error")
        raise
```


```
def _get_collection():
    client = _get_mongo_client()
    return client[MONGO_DB][MONGO_COLLECTION]

```

```
# ---------- Cached reads (simple) ----------
# NOTE: lru_cache here caches results per-process until code reload. If you want TTL-based caching,
# use frappe.cache() or a Redis/bench-backed solution.
@lru_cache(maxsize=1)
def get_mongo_chart_data() -> Dict[str, Any]:
    """
    Aggregates average price per suburb and returns top 6 results formatted for charts.
    Cached per-process for performance — clear by restarting worker or removing cache decorator.
    """
    try:
        coll = _get_collection()
        pipeline = [
            {"$match": {"price": {"$exists": True}}},
            {
                "$group": {
                    "_id": "$address.suburb",
                    "average_price": {"$avg": {"$toDouble": "$price"}}
                }
            },
            {"$sort": {"average_price": -1}},
            {"$limit": 6}
        ]
        rows = list(coll.aggregate(pipeline))
        labels = [(r["_id"] if r.get("_id") else "Unknown") for r in rows]
        values = [round((r.get("average_price") or 0), 2) for r in rows]
        return {"labels": labels, "datasets": [{"name": "Avg Price (USD)", "values": values}]}
    except Exception:
        frappe.log_error(frappe.get_traceback(), "Chart Data Error")
        return {"error": "Failed to load chart data."}


```

```
@lru_cache(maxsize=1)

def get_airbnb_listing_list(limit: int = 20) -> List[Dict[str, Any]]:
    """
    Returns a small list of listings for the demo UI.
    Cached per-process.
    """
    try:
        coll = _get_collection()
        cursor = coll.find(
            {"address.market": {"$exists": True}, "price": {"$exists": True}},
            {"name": 1, "price": 1, "address": 1}
        ).limit(limit)

        result = []
        for doc in cursor:
            result.append({
                "name": doc.get("name", "Unnamed"),
                "price": str(doc.get("price", "N/A")),
                "address": (doc.get("address") or {}).get("market", "Unknown")
            })
        return result
    except Exception:
        frappe.log_error(frappe.get_traceback(), "Listing Load Error")
        return []

```

```
# ---------- Demo context builders & renderers ----------
def get_demo_context_data(filters_json: Optional[str] = None) -> Dict[str, Any]:
    """
    Build a context dict used by both API and rendering functions.
    """
    context = {
        "chart_data": {},
        "listings": [],
        "search_filter": filters_json,
        "search_results": None,
        "search_error": None
    }

    # use the cached readers above
    try:
        chart = get_mongo_chart_data() or {}
        context["chart_data"] = chart
    except Exception:
        frappe.log_error(frappe.get_traceback(), "Chart Load Failed")

    try:
        context["listings"] = get_airbnb_listing_list() or []
    except Exception:
        frappe.log_error(frappe.get_traceback(), "Listings Load Failed")

    if filters_json:
        try:
            filters = json.loads(filters_json)
            result = search_entity(filters=filters)
            if isinstance(result, dict) and "error" in result:
                context["search_error"] = result["error"]
            else:
                context["search_results"] = result
        except Exception as e:
            context["search_error"] = str(e)

    return context

```
```
@frappe.whitelist()
def fetch_demo_page_data(filters: Optional[str] = None) -> Dict[str, Any]:
    if frappe.session.user == "Guest":
        frappe.throw(_("Login required"))

    data = get_demo_context_data(filters)
    data["user"] = frappe.session.user
    return data

# @frappe.whitelist()
# def render_demo_html(filters: Optional[str] = None) -> Dict[str, str]:
#     context = get_demo_context_data(filters)
#     try:
#         html = frappe.render_template("report_app_frappe/templates/pages/demo.html", context)
#         return {"html": html}
#     except Exception:
#         frappe.log_error(frappe.get_traceback(), "Template Render Error")
#         return {"html": "<div class='text-danger'>Failed to render page. Contact admin.</div>"}


# report_app_frappe/report_app_frappe/api/mongo_demo.py
import json
import frappe
from frappe import _
from frappe.utils import cint

# ... (other imports and helper functions stay the same) ...


```

```
@frappe.whitelist()
def render_demo_html(filters=None):
    """
    Render the demo template and also return the context as JSON-serializable
    data so the client can consume chart/listing data without parsing the HTML.
    """
    # Build context using your existing helper (which already converts BSON types)
    context = get_demo_context_data(filters_json=filters)

    # Ensure context is JSON serializable:
    # If convert_bson_types already applied in get_demo_context_data, this is redundant.
    try:
        # safe dump/loads roundtrip to ensure data is JSON-safe (avoids ObjectId/Decimal issues)
        safe_context = json.loads(json.dumps(context, default=str))
    except Exception:
        # fallback: convert to strings for safety
        safe_context = {k: (str(v) if not isinstance(v, (dict, list, int, float, type(None))) else v)
                        for k, v in context.items()}

    try:
        html = frappe.render_template("report_app_frappe/templates/pages/demo.html", context)
    except Exception:
        frappe.log_error(frappe.get_traceback(), "Template Render Error")
        html = "<div class='text-danger'>Failed to render page. Contact admin.</div>"

    # Return both html and the safe JSON context
    return {"html": html, "context": safe_context}

```