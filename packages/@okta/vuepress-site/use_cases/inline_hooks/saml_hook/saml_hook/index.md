---
title: SAML Assertion Inline Hook
excerpt: Customize SAML assertions returned by Okta.
---

# SAML Assertion Inline Hook

<ApiLifecycle access="ea" />

This page provides reference documentation for:

- JSON objects contained in the outbound request from Okta to your external service

- JSON objects you can include in your response

This information is specific to the SAML Assertion Inline Hook, one type of inline hook supported by Okta.

## See Also

For a general introduction to Okta inline hooks, see [Inline Hooks](/use_cases/inline_hooks/).

For information on the API for registering external service endpoints with Okta, see [Inline Hooks Management API](/docs/api/resources/inline-hooks).

For steps to enable this inline hook, see below, [Enabling a SAML Assertion Inline Hook](#enabling-a-saml-assertion-inline-hook).

## About

This type of inline hook is triggered when Okta generates a SAML assertion in response to an authentication request. Before sending the SAML assertion to the service provider that will consume it, Okta calls out to your external service. Your external service can respond with commands to add attributes to the assertion or modify its existing attributes.

This functionality can be used to add data to assertions, including data that is sensitive, calculated at runtime, or complexly-structured and not appropriate for storing in Okta user profiles. Data added this way is never logged or stored by Okta. As an example, SAML assertions generated for a medical app could be augmented with confidential patient data provided by your external service and not stored in Okta.

This inline hook works only when using custom SAML apps, not apps from the OIN.

## Objects in the Request from Okta

The outbound call from Okta to your external service provides you with the contents of the SAML assertion that was generated, which you will be able to augment or modify by means of the commands you return, as well as some contextual information about the authentication request.

Because SAML is XML-based, while the call from Okta to your service uses a JSON payload, the contents of the SAML assertion are converted to a JSON representation.

### data.assertion.subject

Provides a JSON representation of the subject of the SAML assertion. The following is an example of how an XML `<saml:Subject>` element is represented in JSON in this object:

```json
{
  "subject":{  
            "nameId":"administrator1@example.net",
            "nameFormat":"urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified",
            "confirmation":{  
               "method":"urn:oasis:names:tc:SAML:2.0:cm:bearer",
               "data":{  
                  "recipient":"http://www.example.com/saml/sso"
               }
}
``` 
### data.assertion.authentication



### data.assertion.conditions

### data.assertion.claims

Provides a JSON representation of the `<saml:AttributeStatement>` element contained in the in the generated SAML assertion, which will contain any optional SAML attribute statements that you have defined for the app using the Okta Admin Console's **SAML Settings**.

The following is an example of how an XML `<saml:AttributeStatement>` element is represented in JSON in this object:

```json
{
  "claims": {
		"foobie": {
			"attributes": {
				"NameFormat": "urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified"
			},
			"attributeValues": [{
				"attributes": {
					"xsi:type": "xs:string"
				},
				"value": "doobie"
			}]
		}
	}
}
```
### data.assertion.lifetime

### data.context

This object contains a number of sub-objects, each of which provides some type of contextual information. Unlike the `data.assertion.*` objects, you cannot affect the `data.context.*` objects by means of the commands you return. The following sub-objects are included:

 - `data.context.request`: Details of the SAML request that triggered the generation of the SAML assertion.
 - `data.context.protocol`: Details of the assertion protocol being used.
 - `data.context.session`: Details of the user session.
 - `data.context.user`: Identitifes the Okta user that the assertion was generated to authenticate, and provides details of their Okta user profile.

## Objects in Response You Send

For the Token Inline hook, the `commands` and `error` objects that you can return in the JSON payload of your response are defined as follows:

### commands

The `commands` object is where you can provide commands to Okta. It is where you can tell Okta to add additional claims to the assertion or to modify the existing assertion statements.

The `commands` object is an array, allowing you to send multiple commands. In each array element, there needs to be a `type` property and `value` property. The `type` property is where you specify which of the supported commands you wish to execute, and `value` is where you supply an operand for that command.

In the case of the Token hook type, the `value` property is itself a nested object, in which you specify a particular operation, a path to act on, and a value.

| Property | Description                                                              | Data Type       |
|----------|--------------------------------------------------------------------------|-----------------|
| type     | One of the [supported commands](#supported-commands).                    | String          |
| value    | Operand to pass to the command. It specifies a particular op to perform. | [value](#value) |

#### Supported Commands

The following command is supported for the SAML Assertion Inline Hook type:

| Command                 | Description             |
|-------------------------|-------------------------|
| com.okta.assertion.patch | Modify a SAML assertion.     |

#### value

The `value` object is where you specify the specific operation to perform. It is an array, allowing you to request more than one operation.

| Property | Description                                                                                                                                                                                                           | Data Type       |
|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------|
| op       | The name of one of the [supported ops](#list-of-supported-ops).                                                                                                                                                       | String          |
| path     | Location, within the assertion, to apply the operation, specified as a slash-delimited path. When adding a claim, this will always begin with `/claims/` and be followed by the name of the new claim you are adding. | String          |
| value    | Value to set the claim to.                                                                                                                                                                                            | Any JSON object |

#### List of Supported Ops

| Op      | Description                             |
|---------|-----------------------------------------|
| add     | Add a new claim to the assertion.       |
| replace | Modify an existing attribute statement. |

### error

When you return an error object, it should have the following structure:

| Property     | Description                          | Data Type |
|--------------|--------------------------------------|-----------|
| errorSummary | Human-readable summary of the error. | String    |

Returning an error object will cause Okta to return an OAuth 2.0 error to the requester of the token, with the value of `error` set to `server_error`, and the value of `error_description` set to the string you supplied in the `errorSummary` property of the `error` object you returned.

## Sample Listing of JSON Payload of Request

```json
{  
   "eventTypeVersion":"1.0",
   "cloudEventVersion":"0.1",
   "eventType":"com.okta.saml.tokens.transform",
   "contentType":"application/json",
   "source":"https://{yourOktaDomain}/app/raincloud59_saml20app_1/exkth8lMzFm0HZOTU0g3/sso/saml",
   "eventId":"Nb7xMhjcR9eFpHLxJ-rcKg",
   "eventTime":"2019-01-11T20:12:41.000Z",
   "data":{  
      "context":{  },
      "assertion":{  
         "subject":{  },
         "authentication":{  
            "sessionIndex":"id1547237557267.1495952828",
            "authnContext":{  
               "authnContextClassRef":"urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport"
            }
         },
         "conditions":{  },
         "claims":{  
            "foobie":{  
               "attributes":{  
                  "NameFormat":"urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified"
               },
               "attributeValues":[  
                  {  
                     "attributes":{  
                        "xsi:type":"xs:string"
                     },
                     "value":"doobie"
                  }
               ]
            },
            "array":{  
               "attributes":{  
                  "NameFormat":"urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified"
               },
               "attributeValues":[  
                  {  
                     "attributes":{  
                        "xsi:type":"xs:string"
                     },
                     "value":"Array 1"
                  },
                  {  
                     "attributes":{  
                        "xsi:type":"xs:string"
                     },
                     "value":"Array2"
                  },
                  {  
                     "attributes":{  
                        "xsi:type":"xs:string"
                     },
                     "value":"Array3"
                  }
               ]
            },
            "middle":{  
               "attributes":{  
                  "NameFormat":"urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified"
               },
               "attributeValues":[  
                  {  
                     "attributes":{  
                        "xsi:type":"xs:string"
                     },
                     "value":"middellllll"
                  }
               ]
            },
            "firstAndLast":{  
               "attributes":{  
                  "NameFormat":"urn:oasis:names:tc:SAML:2.0:attrname-format:unspecified"
               },
               "attributeValues":[  
                  {  
                     "attributes":{  
                        "xsi:type":"xs:string"
                     },
                     "value":"Add-MinOCloudy Tud"
                  }
               ]
            }
         },
         "lifetime":{  
            "expiration":300
         }
      }
   }
}
```

## Sample Listing of JSON Payload of Response

```json
{  
   "error":null,
   "commands":[  
      {  
         "type":"com.okta.assertion.patch",
         "value":[  
            {  
               "op":"replace",
               "path":"/claims/array/attributeValues/1/value",
               "value":"replacementValue"
            },
            {  
               "op":"replace",
               "path":"/authentication/authnContext",
               "value":{  
                  "authnContextClassRef":"Something:different?"
               }
            },
            {  
               "op":"add",
               "path":"/claims/foo",
               "value":{  
                  "attributes":{  
                     "NameFormat":"urn:oasis:names:tc:SAML:2.0:attrname-format:basic"
                  },
                  "attributeValues":[  
                     {  
                        "attributes":{  
                           "xsi:type":"xs:string"
                        },
                        "value":"barer"
                     }
                  ]
               }
            },
            {  
               "op":"replace",
               "path":"/conditions/audienceRestriction",
               "value":[  
                  "urn:example:sp",
                  "one:element",
                  "two:elements"
               ]
            }
         ]
      },
      {  
         "type":"com.okta.assertion.patch",
         "value":[  
            {  
               "op":"replace",
               "path":"/authentication/sessionIndex",
               "value":"definitelyARealSession"
            }
         ]
      }
   ]
}
```
## Enabling a SAML Assertion Inline Hook

To activate the inline hook, you first need to register your external service endpoint with Okta using the [Inline Hooks Management API](/docs/api/resources/inline-hooks).

You then need to associate the registered inline hook with a Custom Authorization Server Policy Rule by completing the following steps in Admin Console:

1. Go to **Security > API Authorization Servers**.

1. Select the Custom Authorization Server to use this inline hook with.

1. One of the rules defined in the Custom Authorization server needs to be used to trigger invocation of the inline hook. Click the pencil icon for that rule to open it for editing.

1. In the **Advanced Settings** section, click the **Assertion Inline Hook** dropdown menu. Any inline hooks you have registered will be listed. Select the one to use.

1. Click **Update Rule**.

> Note: Only one inline hook can be associated with each rule.
