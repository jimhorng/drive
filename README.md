# drive
The following describes the drive hypermedia specification.  The intent of the drive media type is to provide a standard resource structure that can drive all behavior of a calling application.

The media type for drive JSON is application/vnd.drive+json.  Support for XML may be supported as well.

# Basic Design
All resources contain four optional high level objects
* links
* data
* entities
* schema

# Links
Links provide a list of the API's to related resources. Links must be an array of values which may be empty, but cannot be null.  The following is a list of attributes that defines a Link.
* rel - required - The relation type of the Link
* href - required - The URI for the API
* text - optional - The Localized name of the Link to display.
* title - optional - The descriptive Localized name of the Link.
* method - optional - defaults to GET - the HTTP verb that should be used to invoke the Link.
* type - optional - Specifies the content type that should be used for the payload sent to the Link URI.

**Example:**
```JSON
"links" : [
    {
        "rel": "self",
        "href": "https://api.domain.com/resource",
        "title": "Resource"
    },
    {
        "rel": "edit",
        "href": "https://api.domain.com/resource",
        "title": "Update",
        "method":"PUT",
        "type":"appliation/json"
   } 
]
```

# Data
The data section contains all of the attributes and their values for the resource itself. The data section must be an object which may be empty, but cannot be null.

**Example:**
```JSON
"data": {
	"company": "acmeinc",
	"name": "acme",
	"createdDate": "2015-12-25T13:10:37"
}
```

# Entities
The entities section contains one or more embedded resources.  The entities section must be an object which is optional and may be empty, but cannot be null. The entity structure from here is nested as seen in the example below.

**Example:**
```JSON
"entities": {
	"child" : {
		"links": [],
		"data": {},
		"entities": {},
		"schema": {}
	},
	"children" : [
		{
			"links": [],
			"data": {},
			"entities": {},
			"schema": {}
		}
	]
}
```

# Schema
The JSON schema that defines how the content should be structured for requests.  The schema is optional, but cannot  be null. If provided, the schema must be an object.  Typically the data section would contain properties that then would be described by the schema.  The schema would then be used to build the request content to invoke a provided Link that has a type attribute defined.  The schema should be defined as specified by http://json-schema.org/.

**Example:**
```JSON
"schema": {
	"title": "Company",
	"type": "object",
	"properties": {
		"company": {
			"type": "string"
		},
		"name": {
			"type": "string",
			"pattern":"[a-z,A-Z]"
		},
		"createdDate": {
			"type": "string"
		}
	},
	"required": ["name"]
}
```

# Examples

The following is an example drive represetntation that would allow an application to maintain the details and prices for a product.  This example incorporates the use of two resources.  The first is the product with the second being the prices associated with the product.

Looking at the two resources in the application/json format, the Product and Price would look like below.

| **URL:** /products/{id} |
| **Allowed Methods:** GET & PUT |
```JSON
{
	"name":"Adventure Game",
	"sku":"XYZ",
	"createdDate":"2015-12-25"
}
```

| **URL:** /products/{id}/prices |
| **Allowed Methods:** GET & POST |
```JSON
[
	{
		"amount":10.0,
		"currency":"USD"
	},
	{
		"amount":12.0,
		"currency":"CAD"
	}
[
```

| *URL:* /products/{id}/prices/{id} |
| *Allowed Methods:* GET & PUT |
```JSON
{
	"name":"Adventure Game",
	"sku":"XYZ",
	"createdDate":"2015-12-25"
}
```

The above resources can then be combined into a single Product resource using the drive representation.  All of the valid actions and business rules are provided as shown in the following example.  The product can be edited, but can not be deleted.  The Prices can be created, edited, and deleted.  The currency for the price is defined when the price is created, but the currency cannot be edited. 
```JSON
{
	"links": [
	    {
	        "rel": "self",
	        "href": "https://api.domain.com/products/123",
	        "title": "Product"
	    },
	    {
	        "rel": "up",
	        "href": "https://api.domain.com/products",
	        "title": "Cancel"
	    },
	    {
	        "rel": "edit",
	        "href": "https://api.domain.com/products/123",
	        "title": "Save",
	        "text":"Save product details",
	        "method":"PUT",
	        "type":"application/json"
	    }
	],
	"data": {
		"name":"Adventure Game",
		"sku":"XYZ",
		"createdDate":"2015-12-25"
	},
	"entities": {
		"prices": {
			"links": [
			    {
			        "rel": "prices",
			        "href": "https://api.domain.com/products/123/prices",
			        "title": "Prices"
			    },
			    {
			        "rel": "create",
			        "href": "https://api.domain.com/products/123/prices",
			        "title": "New",
			        "text":"Create product price",
			        "method":"POST",
			        "type":"application/json"
			    }
			],
			"entities": {
				"price": [
					{
						"links": [
						    {
						        "rel": "price",
						        "href": "https://api.domain.com/products/123/prices/USD",
						        "title": "Price"
						    },
						    {
						        "rel": "edit",
						        "href": "https://api.domain.com/products/123/prices/USD",
						        "title": "Save",
						        "text":"Edit product price",
						        "method":"PUT",
						        "type":"application/json"
						    },
						    {
						        "rel": "delete",
						        "href": "https://api.domain.com/products/123/prices/USD",
						        "title": "Delete",
						        "text":"Remove the product price",
						        "method":"DELETE"
						    }
						],
						"data": {
							"amount":10.0,
							"currency":"USD"
						},
						"schema": {
							{
							  "title": "Price",
							  "type": "object",
							  "properties": {
							    "amount": {
							      "type": "number",
							      "default":"0.0"
							    },
							    "currency": {
							      "readonly: true,
							      "type": "string",
							      "enum": [
							        "USD",
							        "CAD",
								"EUR"
							      ]
							    }
							  },
							  "required":["amount"]
							}
						}
					},
					{
						"links": [
						    {
						        "rel": "price",
						        "href": "https://api.domain.com/products/123/prices/CAD",
						        "title": "Price"
						    },
						    {
						        "rel": "edit",
						        "href": "https://api.domain.com/products/123/prices/CAD",
						        "title": "Save",
						        "text":"Edit product price",
						        "method":"PUT",
						        "type":"application/json"
						    },
						    {
						        "rel": "delete",
						        "href": "https://api.domain.com/products/123/prices/CAD",
						        "title": "Delete",
						        "text":"Remove the product price",
						        "method":"DELETE"
						    }
						],
						"data": {
							"amount":12.0,
							"currency":"CAD"
						},
						"schema": {
							{
							  "title": "Price",
							  "type": "object",
							  "properties": {
							    "amount": {
							      "type": "number",
							      "default":"0.0"
							    },
							    "currency": {
							      "readonly: true,
							      "type": "string",
							      "enum": [
							        "USD",
							        "CAD",
								"EUR"
							      ]
							    }
							  },
							  "required":["amount"]
							}
						}
					}
				]
			},
			"schema": {
				{
				  "title": "Price",
				  "type": "object",
				  "properties": {
				    "amount": {
				      "type": "number",
				      "default":"0.0"
				    },
				    "currency": {
				      "type": "string",
				      "enum": [
				        "USD",
				        "CAD",
					"EUR"
				      ]
				    }
				  },
				  "required":["amount","currency"]
				}
                        }
		}
	},
	"schema": {
		"title": "Product",
		"type": "object",
		"properties": {
			"name": {
				"type": "string",
				"pattern":"[a-z,A-Z]"
			},
			"sku": {
				"type": "string"
			},
			"createdDate": {
				"type": "string",
				"readonly": true
			}
		},
		"required": ["name","sku"]
	}
}
```
