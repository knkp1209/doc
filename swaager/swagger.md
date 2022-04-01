```json
{
  "openapi": "3.0.0",
  "servers": [
    {
      "url": "http://petstore.swagger.io/v2"
    }
  ],
  "info": {
    "description": ":dog: :cat: :rabbit: This is a sample server Petstore server.  You can find out more about Swagger at [http://swagger.io](http://swagger.io) or on [irc.freenode.net, #swagger](http://swagger.io/irc/).  For this sample, you can use the api key `special-key` to test the authorization filters.",
    "version": "1.0.0",
    "title": "Swagger Petstore",
    "termsOfService": "http://swagger.io/terms/",
    "contact": {
      "email": "apiteam@swagger.io"
    },
    "license": {
      "name": "Apache 2.0",
      "url": "http://www.apache.org/licenses/LICENSE-2.0.html"
    }
  },
  "tags": [
    {
      "name": "temp",
      "description": "Everything about your Pets",
      "externalDocs": {
        "description": "Find out more",
        "url": "http://swagger.io"
      }
    }
  ],
  "paths": {
    "/super/product-group/create-qrcode-preview": {
      "get": {
        "tags": [
          "temp"
        ],
        "summary": "接口名称",
        "description": "备注",
        "parameters": [
          {
            "name": "status",
            "in": "query",
            "description": "字段说明",
            "required": true
          },
          {
            "name": "test",
            "in": "query",
            "description": "字段说明",
            "required": true
          }
        ],
        "responses": {
          "200": {
            "description": "successful operation",
            "content": {
              "application/xml": {
                "schema": {
                  "type": "array",
                  "items": {
                    "type": "string",
                    "enum": [
                      "available",
                      "pending",
                      "sold"
                    ],
                    "default": "available"
                  }
                }
              },
              "application/json": {
                "schema": {
                  "type": "string",
                  "enum": [
                    "available",
                    "pending",
                    "sold"
                  ],
                  "default": "available"
                }
              }
            }
          },
          "400": {
            "description": "Invalid status value"
          }
        },
        "security": [
          {
            "petstore_auth": [
              "write:pets",
              "read:pets"
            ]
          }
        ]
      },
      "post": {
        "tags": [
          "temp"
        ],
        "summary": "接口名称",
        "description": "备注",
        "requestBody": {
          "content": {
            "application/json": {
              "schema": {
                "type": "array",
                "title": "sku_ids",
                "description": "sku_id 数组",
                "items": {
                  "type": "number"
                }
              }
            }
          }
        },
        "responses": {
          "200": {
            "description": "successful operation",
            "content": {
              "application/xml": {
                "schema": {
                  "description": "aa",
                  "type": "array",
                  "items": {
                    "type": "string",
                    "description": "aa",
                    "enum": [
                      "available",
                      "pending",
                      "sold"
                    ],
                    "default": "available"
                  }
                }
              },
              "application/json": {
                "schema": {
                  "type": "string",
                  "description": "bb",
                  "enum": [
                    "available",
                    "pending",
                    "sold"
                  ],
                  "default": "available"
                }
              }
            }
          },
          "400": {
            "description": "Invalid status value"
          }
        },
        "security": [
          {
            "petstore_auth": [
              "write:pets",
              "read:pets"
            ]
          }
        ]
      }
    }
  },
  "externalDocs": {
    "description": "See AsyncAPI example",
    "url": "https://mermade.github.io/shins/asyncapi.html"
  },
  "security": []
}
```

