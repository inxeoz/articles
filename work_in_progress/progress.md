

### 1. Exploring Frappe

> [Doctype](exploring_frappe/doctype.md)

>[Workflow](exploring_frappe/workflow.md) 

>[Role / Role Permission](exploring_frappe/role_and_role_permission.md)

>[Client and Server Script](exploring_frappe/client_and_server_script.md)

### 2. Implement VPMS Flow In Frappe

>Creation of Users / Role i.e CGM, Marketing Team, PO, Managing Director etc.

> Creating Bid Creation Document

>Creating Bid Document Status i.e Draft, Pending, Approved

>Creation Of DocType like Response Sheet,  Corrigendum, Member etc.

>Implementation of VPMS Flow


### 3. Reports in Frappe Using Mongodb

> [Installing MongoDb Sample Data](reports_in_frappe/installing_sample_data.md)
>> https://github.com/neelabalan/mongodb-sample-dataset.git

>[Creating Virtual DocType named "Airbnb data"](reports_in_frappe/creating_virtual_doctype_airbnb_data.md)
> > to Show the Records of Airbnb data from mongodb to frappe

> [Creating Frappe API To Crunch and Serve Report Data](reports_in_frappe/frappe_api_for_data_crunch_serve.md)
> >report_app_frappe.report_app_frappe.api.token_utils import verify_token


> Creating templates / pages / demo.html (jinja template)

>  demo.py will call the Frappe API and bind response with html jinja
> >context.update(get_demo_context_data(filters_json))

> demo page access-able 


### 4. Finding Ways to Integrating Frappe Into Frameworks Like React
> Possible Solutions
> > 1. iframe embedding
> > 2. rest-api of frappe
> > 3. web page  / pages ( still using iframe )

> Iframe is a risky Way To Integrate (So Removed from list )

> Rest-Api in react

### 5. Login Page In React Using Frappe Rest-Api

>Ways to Login 
> >1. User id and Password
> >2. Auth Token
>
> returns cookies 

> Using Auth Token To access

#### `only logged in user can see the report`
> 1. Using Auth Token
> 2. Frappe Backend Verifies Token
> 3. Verified => let access report

### 6. UI Components of frappe as Rest Api ?

> Have Not Found any solution

### 7.  JSON Format response of Doctype, DocType Records

>We can get Json response of DocType and its records and other 

>essentially all the service provide by frappe responds in JSON

### 8.  React in Frappe (`Opposite of Stage 4`)

>Understand `monday.com` IT support dashboard and workflow
>>1. Its a (user / dev. query ticket) managing workflow
>>2. Have Other Features like gantt chart , kanban , calendar etc.

>Development
>>1. Create Own Version of IT Support inspired By `monday.com` in react
>>2. Build Static Pages for `IT Support React App`
>>3. Create a Web Page DocType `own Page`
>>4. fetch Static Build scripts / html / css in runtime and load in `Own Page`

>Deploy
>> 1. Deploying in Aws Ec2 Ubuntu instance
>> 2. setup of nginx

### 9. integration of frappe-wiki into react

>We Concluded that Whole Wiki integration is not possible into React `Stage 4`
> We Have Options
> >1. Rest Api => Have To Create Own Ui And Logic
>
>>2. Redirect From React to Frappe-Wiki with token
>
> >Going With Option 2 (`Redirect From React To Wiki`)
>>
> >1. React App (`admin`) will Create access list for pages (edit , visible) and save this into database and generate a unique token for that access list
> >2. Redirect User With Token to Wiki
> >3. Wiki will verify the token and fetch the access list from same database 
> >4. Based on fetched access list wiki backend will create sidebar 
> >5. Based on fetched access list wiki backend will control access of pages and editing permission

>Working
> >1. `To remove Guest Access control` (Enabling Guest Access allows all the document space to be visible , we don't want that to happen)
> >2. Implementing `Token based access` (editable , visible)
> >3. `Fixing Bug` (On Direct Access,  Navigation Button (Prev, next) is not visible)

>Next
> >1. Feature to download documents into pdf / word formate
> >2. if no token provided => consider as public access (we have to implement public access as well)
> >3. Binding token with Ip address


