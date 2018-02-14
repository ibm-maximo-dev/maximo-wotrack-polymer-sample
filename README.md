# Maximo Work Orders Polymer Application

Use the Polymer sample to learn how you can use Maximo application programming interface (APIs) with a web application. This sample uses APIs to perform work order operations.

If you are familiar with Polymer and want to get started with the new release, you're in the right place! If you'd like an introduction to the Polymer project and web components, check [here](https://www.polymer-project.org/2.0/start/).

This document is not intended to be a substitute for the official Maximo API documentation. If any contradiction is present, you must consider the official documentation as the source of truth.

You must have a basic knowledge about web development, mainly JavaScript, Polymer and XML HTTP requests. 

## Prerequisites

- [Polymer 2](https://www.polymer-project.org/) Installed (check the instructions [here](https://www.polymer-project.org/2.0/start/install-2-0))

- A Maximo instance running that has the following system properties defined:
  -  `mxe.oslc.aclalloworigin` defined with the URL or IP address of the Polymer application.
  - `mxe.oslc.aclallowheaders` defined with `maxauth,x-method-override,patchtype,content-type,accept,x-public-uri, properties`

The sample application was built and tested using Google Chrome Version 63.

### How to Define Values to System Properties
  
To define system properties, log in to Maximo and go to *System Properties* application (Open the main menu and select `System Configuration` > `Platform Configuration` > `System Properties`), This will open the *System Properties* application. Click in the *Filter* link on top of *Global Properties* table, type *mxe.oslc.aclallow* on *Property Name* field and press *Enter/Return* key, the table will show the CORS related properties. 
  
To change the value of each property, expand the property (click on *View Details* button, a gray right arrow next to the checkbox) and change its *Global Value* field. Perform this for each variable then save them (click on the *Save* button on the top of the screen)

After saving the properties, select them, clicking on their checkboxes, and click *Live Refresh*, on *Common Actions* in the left menu.

## Getting started

1. Clone this repository.

2. Run the following commands on a terminal:
```
polymer install
polymer serve
```

**Note:** To enable Polymer server to be remotely accesssible, a binding to the local host should be used: `polymer serve --hostname 0.0.0.0`

3. The console will print a localhost URL of the server hosting the application, open it in a browser to load the login screen. The URL should look similar to this: [http://127.0.0.1:8081](http://127.0.0.1:8081), but the port number may be different.

4. If everything worked, the browser should present a login screen with three fields: `Username`, `Password` and `Maximo URL`, the first two should be filled with a valid maximo user credential and the last one the base URL of the Maximo instance to where the validation should be sent, be sure to include the protocol part(*HTTP://* or *HTTPS://*) on the beginning of the URL.

5. After a successful login, the application should open the Work Order Listing page.

## Features

- **Log in:** The user is able to login using Maximo credentials.
- **List Work Orders:** A list of Work Orders is presented in a paged list.
- **Work Order details:** It is possible to see detailed info of a single Work Order.
- **Update Work Order details:** The user can update information of a Work Order
- **Create new Work Order:** The user can create new Work Orders

## How to use the web elements

### Authentication

You need to authenticate before you can do any other action. Send the authentication request to the `maximo/oslc/login` endpoint with a `maxauth` request header that has a base64 encoded string in the form "userid:password". This authentication can be done by using the [iron-ajax](https://www.webcomponents.org/element/PolymerElements/iron-ajax) web component:

```html
<iron-ajax
    id="maximoLoginAjax"
    with-credentials
    content-type="application/json"
    url="[[baseUrl]]maximo/oslc/login"
    method="post"
    handle-as="json"
    last-response="{{loginResponse}}"></iron-ajax>
```

The above example shows a regular `iron-ajax` element that performs a native authentication, but before sending this request by calling its `generateRequest` method, include the `maxauth` header:

```javascript
performLogin() {
    //maximoLoginAjax - id of the iron-ajax component
    this.$.maximoLoginAjax.headers['maxAuth'] = btoa(`${this.get('username')}:${this.get('password')}`)
    this.$.maximoLoginAjax.generateRequest()
}
```

The [btoa](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/btoa) function creates a base-64 encoded ASCII string that is based on its input string, and it is used to generate the value of the header as the specification defines.

The `with-credentials` flag must be set to true in all the requests that are made, so the JavaScript engine can store the authentication session on a cookie and reuse it on subsequent requests. Read more about this mechanism [here](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/withCredentials). The credentials are invalidated only after they expire or the user logs out.

Check the full sample for more information: [src/maximo-login.html](src/maximo-login.html)

The authentication mechanism creates a session that is controlled by the Maximo server. To end the session you need to send a `POST` request to the `maximo/oslc/logout` endpoint, which is shown in the following example:

```html
<iron-ajax
    auto
    with-credentials
    content-type="application/json"
    url="[[baseUrl]]maximo/oslc/logout"
    method="post"
    handle-as="json"
    last-response="{{logoutResponse}}"></iron-ajax>
```

The logout link on the Sample application redirects to a page that contains the above `iron-ajax` element, and the `auto` attribute forces execution after the page loads.

Check the full sample for more information: [src/maximo-logout.html](src/maximo-logout.html)

Sometimes you need to know if the session is still active or not, you can achieve that consulting the `maximo/oslc/whoami` endpoint:
```html
<iron-ajax
    auto
    with-credentials
    content-type="application/json"
    url="[[baseUrl]]maximo/oslc/whoami"
    method="get"
    handle-as="json"
    content-type="application/json"
    last-response="{{whoAmI}}"></iron-ajax>
```

That mechenism is used on the sample to define if a user is still authenticated after a page reload. Check the full sample for more information: [src/maximo-app.html](src/maximo-app.html)

### Retrieve Resource Set

After authentication, it's possible to use all endpoints of the API. The endpoints are grouped by resources. 
One of the available resources is work orders. The following example shows a regular `iron-ajax` element that retrieves all work orders:

```html
<iron-ajax
    auto
    with-credentials
    content-type="application/json"
    url="[[baseUrl]]maximo/oslc/os/mxwo"
    params="[[_urlParams(pageData.num)]]"
    method="get"
    handle-as="json"
    last-response="{{workOrders}}"></iron-ajax>
```

The `mxwo`attribute is the identifier of the work order resource, and if you replace it with `mxasset`, you will get all assets. Check the API documentation to see all the available identifiers.

The `_urlParams` function is responsible for creating the query parameters that can be sent to the endpoint:

```javascript
_urlParams(pageNum) {
    return {
        'oslc.pageSize': 100, //the number of records on a single page
        'oslc.select': "wonum,description,name,status,status_description, workorderid", //the fields that will be returned on the response
        'collectioncount': 1, //also returns the total number of records and pages
        'oslc.where': 'spi:istask=0 and spi:status="WAPPR"', //filter for Work Orders Waiting for approval only
        "pageno": pageNum //Set the requested page number 
    }
}
```
The above example specifies that the query will return 100 records per page, with only `wonum,description,name,status,status_description, workorderid` fields in each record. The`pageno` attribute defines the current page, so you can change the page by using the 'Next Page' and 'Previous Page' links. It is necessary to set `collectioncount` in order to get the total number of records returned by this query.

Check the full sample for more information: [src/maximo-work-orders-list.html](src/maximo-work-orders-list.html)

### Get and Update Resource Details

You can get resource details by performing a `GET` request on the `maximo/oslc/os/<resource name>/<resource id>` endpoint. The resource id is the primary key column name, which is `workorderid` for the `workorder` table:

```html
<iron-ajax
    auto
    with-credentials
    method="get"
    content-type="application/json"
    handle-as="json"
    url="[[baseUrl]]maximo/oslc/os/mxwo/{{pageData.woRef}}"
    last-response="{{workOrder}}"></iron-ajax>
```

You can also update resource details by sending a `POST` request to the same endpoint, which is shown in the following example:

```html
<iron-ajax
    with-credentials
    content-type="application/json"
    url="[[baseUrl]]maximo/oslc/os/mxwo/{{workOrder.spi:workorderid}}"
    method="post"
    handle-as="json"
    last-response="{{workOrder}}"
    headers="[[_getSaveHeaders()]]"
    body="{{_getSavePayload(workOrder)}}"></iron-ajax>
```

The example above uses the global variable `workOrder`, which is populated by the previous load request to the the correct id. The `_getSaveHeaders` function adds the necessary headers to the request: a `x-method-override` that has the value `PATCH` and the optional `properties` header, where you can set the fields that you want to be on the response json (`*` wildcard accepted):

```javascript
_getSaveHeaders() {
    return {
        'x-method-override': 'PATCH',
        'properties': '*'
    }
}
```

The body of the request must have a JSON object with all the fields that must be updated. The following example re-sends the `workOrder` object that was returned by the get resource call:
```javascript
_getSavePayload(workOrder) {
    return workOrder
}
```

Check the full sample for more information: [src/maximo-work-order.html](src/maximo-work-order.html)

## References

For more information, see the following resources:
- [IBM Asset Management Developer Center](https://developer.ibm.com/iot/asset-management/)



© Copyright IBM Corporation 2018.
