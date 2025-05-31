# Invoice Radar Plugin Handbook

## 1. Introduction

Welcome to the Invoice Radar Plugin Handbook for developers!

This guide will help you create custom plugins to fetch invoices and receipts from various platforms.

Invoice Radar is a document automation tool that helps you fetch, download, and organize invoices and receipts from various platforms.

[📟 Learn more about Invoice Radar](https://invoiceradar.com/)

#### Table of Contents

1. [Introduction](#1-introduction)
2. [Getting Started](#2-getting-started)
3. [Plugin Structure](#3-plugin-structure)
4. [Writing Your First Plugin](#4-writing-your-first-plugin)
5. [Useful Patterns](#5-useful-patterns)
6. [Steps Reference](#steps-reference)
7. [Autofill Configuration](#autofill-configuration)

## 2. Getting Started

### Prerequisites

- Basic knowledge of JSON, HTML, CSS and JavaScript.
- A text editor or IDE (e.g., VSCode, Sublime Text).
- Invoice Radar installed on macOS or Windows.

### Installation

1. **Download and Install Invoice Radar**:

   - [Request Access to Invoice Radar](https://invoiceradar.com/)

2. **Download the Blank Plugin**:

   - [Download the Blank Plugin](https://github.com/invoiceradar/plugins/blob/main/blank-plugin.json) to your local machine.
   - Rename the file to `your-plugin-name.json`.
   - Put it into a folder of your choice.

3. **Add the Plugin to Invoice Radar**:
   - Open Invoice Radar.
   - Navigate to settings and choose the `Available Plugins` tab.
   - Choose `Choose Plugin Directory` and select the folder where you saved the plugin.
   - Your plugin should now appear in the list of available plugins.

## 3. Plugin Structure

Plugins for Invoice Radar are written in JSON and follow a specific structure. Each plugin consists of the following sections:

**Plugin Description**:

- **Metadata**: Basic information about the plugin, such as name, description, and homepage URL.
- **configSchema**: Configuration properties that the plugin may require.

**Scraping Steps**:

- **checkAuth**: Steps to verify if the user is already authenticated.
- **startAuth**: Steps to initiate the authentication process.
- **getConfigOptions**: Optional steps to fetch configuration options like Team ID.
- **getDocuments**: Steps to fetch and download documents.

### Minimal Plugin Example

```json
{
  "$schema": "https://raw.githubusercontent.com/invoiceradar/plugins/main/schema.json",
  "id": "example",
  "name": "Example Platform",
  "description": "Short description of the service.",
  "homepage": "https://example.com",
  "checkAuth": [
    {
      "action": "navigate",
      "url": "https://example.com/dashboard"
    },
    {
      "action": "checkElementExists",
      "selector": "#logout-button"
    }
  ],
  "startAuth": [
    {
      "action": "navigate",
      "url": "https://example.com/login"
    },
    {
      "action": "waitForElement",
      "selector": "#account-summary",
      "timeout": 120000
    }
  ],
  "getDocuments": [
    {
      "action": "navigate",
      "url": "https://example.com/billing"
    },
    {
      "action": "extractAll",
      "selector": ".invoice-row",
      "variable": "invoice",
      "fields": {
        "id": {
          "selector": ".invoice-id"
        },
        "date": {
          "selector": ".invoice-date"
        },
        "total": {
          "selector": ".invoice-total"
        },
        "url": {
          "selector": ".invoice-download",
          "attribute": "href"
        }
      },
      "forEach": [
        {
          "action": "downloadPdf",
          "url": "{{invoice.url}}",
          "document": "{{invoice}}"
        }
      ]
    }
  ]
}
```

The full schema can be found [here](https://raw.githubusercontent.com/invoiceradar/plugins/main/schema.json).

## 4. Writing Your First Plugin

### Step-by-Step Guide

Let's create a simple plugin to fetch invoices from a hypothetical service.

1. **Define Metadata**:

   This information is used to identify and display the plugin in Invoice Radar. The homepage URL is used to get the favicon of the service.

   Note that the `id` should be unique and lowercase.

   ```json
   {
     "id": "example-service",
     "name": "Example Service",
     "description": "Short description of the service.",
     "homepage": "https://example.com"
   }
   ```

   [Learn more](#) about metadata fields.

2. **Define Configuration Schema** (Optional):

   The configuration schema defines which fields are required for the plugin to function. In this example, we need a `teamID` and `password` to authenticate.

   These fields will be displayed to the user when adding the plugin in Invoice Radar.

   ```json
   "configSchema": {
     "teamID": {
       "type": "string",
       "title": "Team ID",
       "description": "The ID of your team or account to fetch invoices.",
       "required": true
     }
   }
   ```

   [Learn more](#) about configuration schema fields.

3. **Check Authentication**:

   `checkAuth` contains steps to verify if the user is authenticated. This can be done by checking the URL or element existence. The last step inside `checkAuth` needs to be a [verification step](#✅-verification-steps).

   These steps are executed when a run is started. If the user is already authenticated, the plugin will skip the authentication process and go directly to fetching documents.

   ```json
   "checkAuth": [
     {
       "action": "navigate",
       "url": "https://example.com/dashboard"
     },
     {
       "action": "checkElementExists",
       "selector": "#logout-button"
     }
   ]
   ```

4. **Start Authentication**:

   `startAuth` contains steps to initiate the authentication process. This can involve navigating to the login page and waiting for a successful login indicator.

   **The browser will be visible during** the authentication process, allowing the user to interact with the login form.

   ```json
   "startAuth": [
     {
       "action": "navigate",
       "url": "https://example.com/login"
     },
     {
       "action": "waitForElement",
       "selector": "#account-summary",
       "timeout": 120000
     }
   ]
   ```

5. **Expose Configuration Options (optional)**:

   The `getConfigOptions` step is optional and can be used to fetch configuration options like a teamId. This is useful if the plugin requires a teamId to be set in the configuration which is not known at the time of adding the plugin.

   The `config` field is the key of one of the options in the `configSchema`. It will expose the options to the user as a dropdown in the configuration modal.

   ```json
   "getConfigOptions": [
      {
        "action": "navigate",
        "url": "https://example.com/",
        "waitForNetworkIdle": true
      },
      {
        "action": "extractAll",
        "script": "fetch('https://example.com/api/teams').then(r => r.json())",
        "variable": "option",
        "forEach": [
          {
            "action": "exposeOption",
            "config": "teamId",
            "option": {
              "label": "{{option.name}}",
              "value": "{{option.id}}"
            }
          }
        ]
      }
   ]
   ```

6. **Scrape Documents**:

   `getDocuments` contains steps to fetch and download documents. This can involve navigating to the billing page, extracting invoice details, and downloading the PDFs.

   ```json
   "getDocuments": [
     {
       "action": "navigate",
       "url": "https://example.com/billing"
     },
     {
       "action": "extractAll",
       "selector": ".invoice-row",
       "variable": "invoice",
       "fields": {
         "id": {
           "selector": ".invoice-id"
         },
         "date": {
           "selector": ".invoice-date"
         },
         "total": {
           "selector": ".invoice-total"
         },
         "url": {
           "selector": ".invoice-download",
           "attribute": "href"
         }
       },
       "forEach": [
         {
           "action": "downloadPdf",
           "url": "{{invoice.url}}",
           "document": {
             "type": "invoice",
             "id": "{{invoice.id}}",
             "date": "{{invoice.date}}",
             "total": "{{invoice.total}}"
           }
         }
       ]
     }
   ]
   ```

7. **You are done!**:

   Save the file and add it to Invoice Radar. You can now run the plugin to fetch invoices from the service.

## 5. Useful Patterns

### Common patterns for authentication checks (`checkAuth`)

#### Pattern 1: Go to Dashboard and check URL

Many services automatically redirect to the login page if the user is not authenticated. We can use this behavior to check if the user is authenticated.

```json
{
  "action": "navigate",
  "url": "https://example.com/login"
},
{
  "action": "checkURL",
  "url": "https://example.com/account",
}
```

Depending on the service, they may redirect you from the dashboard to the login page if you are not authenticated. In this case, you can use the `checkURL` step to check if the URL still matches after visiting the dashboard.

```json
{
  "action": "navigate",
  "url": "https://example.com/dashboard"
},
{
  "action": "checkURL",
  "url": "https://example.com/dashboard",
}
```

Note that you can use glob patterns to match dynamic URLs: `https://example.com/dashboard/**`.

#### Pattern 2: Check for Logout Button

You can use a selector that is unique to the authenticated state to check if the user is authenticated, e.g. a logout button or profile link.

```json
{
  "action": "navigate",
  "url": "https://example.com/home"
},
{
  "action": "waitForElement",
  "selector": "#logout-button"
}
```

#### Tip: Make sure the website is fully loaded

In some cases, the website has not fully loaded when the `checkElementExists` step is executed. To avoid this, you can use the `waitForNetworkIdle` attribute to wait for the page to be fully loaded.

```json
{
  "action": "navigate",
  "url": "https://example.com/home",
  "waitForNetworkIdle": true
},
{
  "action": "checkElementExists",
  "selector": "#logout-button"
}
```

### Common patterns for start authentication (`startAuth`)

#### Pattern 1: Go to Login Page and wait for logged in state

Most authentication processes start by navigating to the login page and waiting for a specific element to appear after a successful login.

Remember that the browser will be visible during the authentication process, allowing the user to interact with the login form. The authentication flow itself can be automated, but isn't requried.

```json
{
  "action": "navigate",
  "url": "https://example.com/login"
},
{
  "action": "waitForElement",
  "selector": "#logout-button",
  "timeout": 120000
}
```

To give the user enough time to log in, it's recommend to provide a long timeout to the wait step, with a default of 120 seconds.

## Steps Reference

This section provides an overview of the available steps that can be used to create plugins for Invoice Radar. Each step represents a specific action that can be performed during the automation process.

### Table of Contents

- [🌐 Navigation Steps](#🌐-navigation-steps)

  - [Navigate (`navigate`)](#navigate-navigate)
  - [Wait for URL (`waitForURL`)](#wait-for-url-waitforurl)
  - [Wait for Element (`waitForElement`)](#wait-for-element-waitforelement)
  - [Wait for Navigation (`waitForNavigation`)](#wait-for-navigation-waitfornavigation)
  - [Wait for Network Idle (`waitForNetworkIdle`)](#wait-for-network-idle-waitfornetworkidle)

- [⚡️ Interaction Steps](#⚡️-interaction-steps)

  - [Click Element (`click`)](#click-element-click)
  - [Type Text (`type`)](#type-text-type)
  - [Select Dropdown (`dropdownSelect`)](#select-dropdown-dropdownselect)
  - [Run JavaScript (`runJs`)](#run-javascript-runjs)

- [✅ Verification Steps](#✅-verification-steps)

  - [Check Element Exists (`checkElementExists`)](#check-element-exists-checkelementexists)
  - [Check URL (`checkURL`)](#check-url-checkurl)
  - [Run JavaScript (`runJs`)](#run-javascript-runjs-1)

- [⚙️ Data Extraction Steps](#⚙️-data-extraction-steps)

  - [Extract (`extract`)](#extract-extract)
  - [Extract All (`extractAll`)](#extract-all-extractall)
  - [Extract Network Response (`extractNetworkResponse`)](#extract-network-response-extractnetworkresponse)

- [📄 Document Retrieval Steps](#📄-document-retrieval-steps)

  - [Download PDF (`downloadPdf`)](#download-pdf-downloadpdf)
  - [Wait for PDF Download (`waitForPdfDownload`)](#wait-for-pdf-download-waitforpdfdownload)
  - [Print Page as PDF (`printPdf`)](#print-page-as-pdf-printpdf)
  - [Download Base64 PDF (`downloadBase64Pdf`)](#download-base64-pdf-downloadbase64pdf)

- [🔀 Conditional Logic Steps](#🔀-conditional-logic-steps)

  - [If (`if`)](#if-if)

- [📦 Miscellaneous Steps](#📦-miscellaneous-steps)

  - [Sleep (`sleep`)](#sleep-sleep)

- [✂️ Snippets](#✂️-snippets)
  - [Get Invoice from Stripe URL (`getInvoiceFromStripeUrl`)](#get-invoice-from-stripe-url-getinvoicefromstripeurl)
  - [Get Invoices from Stripe Customer Portal (`getInvoicesFromStripeBillingPortal`)](#get-invoices-from-stripe-customer-portal-getinvoicesfromstripebillingportal)

### 🌐 Navigation Steps

#### Navigate (`navigate`)

Navigates to the given URL and waits for the page to load. By default, it only waits for the initial page load, not for any subsequent AJAX requests.

```json
{
  "action": "navigate",
  "url": "https://example.com"
}
```

You can set `waitForNetworkIdle` to `true` to ensure the page is fully loaded before continuing.

```json
{
  "action": "navigate",
  "url": "https://example.com/dashboard",
  "waitForNetworkIdle": true
}
```

**Good to know**:

- Relative URLs are supported and will be resolved based on the current page.
- The navigate action will only wait for the initial page load, not for any subsequent AJAX requests.

#### Wait for URL (`waitForURL`)

Waits for the current URL to match the given URL, optionally with a timeout. Supports wildcards.

```json
{
  "action": "waitForURL",
  "url": "https://example.com/profile/**",
  "timeout": 3000
}
```

#### Wait for Element (`waitForElement`)

Waits for the given selector to appear on the page, optionally with a timeout.

```json
{
  "action": "waitForElement",
  "selector": "#example",
  "timeout": 3000
}
```

#### Wait for Navigation (`waitForNavigation`)

Waits for the page navigation to happen. This step will not wait for the page to be fully loaded. Use the [waitForNetworkIdle](#wait-for-network-idle-waitfornetworkidle) step for that purpose. Timeout is optional and defaults to 10 seconds

```json
{
  "action": "waitForNavigation",
  "timeout": 10000
}
```

#### Wait for Network Idle (`waitForNetworkIdle`)

Waits for the network to be idle. This is useful if you want to ensure the page has finished loading all resources. The steps completes when there are no more network requests for 500ms. Timeout is optional and defaults to 15 seconds.

The [`navigate`](#navigate-navigate) step has a `waitForNetworkIdle` option that can be set to `true` to get the same behavior.

```json
{
  "action": "waitForNetworkIdle",
  "timeout": 10000
}
```

### ⚡️ Interaction Steps

#### Click Element (`click`)

Clicks the element specified by the given selector on the page.

```json
{
  "action": "click",
  "selector": "#button"
}
```

#### Type Text (`type`)

Types the given text into the element specified by the given selector on the page.

```json
{
  "action": "type",
  "selector": "#input",
  "value": "Hello World"
}
```

#### Select Dropdown (`dropdownSelect`)

Selects the given value from the dropdown specified by the given selector on the page. The selection happens based on the `value` attribute of the option.

```json
{
  "action": "dropdownSelect",
  "selector": "#dropdown",
  "value": "Option 1"
}
```

#### Run JavaScript (`runJs`)

Runs the given JavaScript in the page context. If a promise is returned, it will be awaited.

If you want to use the result of a script in subsequent steps, use the [extract](#extract-extract) step instead.

```json
{
  "action": "runJs",
  "script": "document.querySelector('#example').click();"
}
```

### ✅ Verification Steps

These steps are used inside `checkAuth` to verify if the user is authenticated.

#### Check Element Exists (`checkElementExists`)

Checks if the given selector exists on the page. Typically used for authentication checks.

```json
{
  "action": "checkElementExists",
  "selector": "#example"
}
```

#### Check URL (`checkURL`)

Checks if the current URL matches the given URL. Supports wildcards patterns like `https://example.com/dashboard/**`.

```json
{
  "action": "checkURL",
  "url": "https://example.com"
}
```

#### Run JavaScript (`runJs`)

The `runJs` step can be used as verification step as well. By running a script that returns a truthy or falsy value, you can verify if the user is authenticated.

```json
{
  "action": "runJs",
  "script": "document.cookie.includes('authToken');"
}
```

### ⚙️ Data Extraction Steps

These steps are used to load data from the page, like a list of items or a single value, and use it in subsequent steps.

#### Extract (`extract`)

Extracts a single piece of data from the page and stores it in a [variable](#).

_Using a CSS fields:_

```json
{
  "action": "extract",
  "variable": "account",
  "fields": {
    "id": "#team-id",
    "name": "#team-name",
    "url": {
      "selector": "#team-link",
      "attribute": "href"
    }
  }
}
```

In this example `account` is used as variable name, and the fields `id`, `name`, and `url` are extracted using CSS selectors. They can be used in subsequent steps using the `{{account.id}}`, `{{account.name}}`, and `{{account.url}}` placeholders.

_Using JavaScript:_

```json
{
  "action": "extract",
  "variable": "token",
  "script": "localStorage.getItem('authToken')"
}
```

This example creates a `token` variable that is extracted using JavaScript. The value can be accessed using the `{{token}}` placeholder. It's also possible to return an object.

#### Extract All (`extractAll`)

Extracts a list of data from the page, and runs the given steps for each item. This is commonly used to iterate over a list of invoices and download them.

For each element matching the `selector`, the fields are extracted and stored in the `variable` available in the `forEach` steps.

**Good to know**:

- Each selector inside the `fields` object is automatically scoped to the matched element.
- The `variable` field is optional. If not provided, the extracted data will be stored in the default variable `item`.
- The current index can be accessed using the `{{index}}` placeholder. It starts at 0 and increments for each item.

_With CSS fields:_

```json
{
  "action": "extractAll",
  "selector": ".invoice-list .invoice-item",
  "variable": "invoice",
  "fields": {
    "id": "td.invoice-id",
    "date": "td.invoice-date",
    "total": "td.invoice-total",
    "url": {
      "selector": "a.invoice-link",
      "attribute": "href"
    }
  },
  "forEach": [
    {
      "action": "navigate",
      "url": "{{invoice.url}}"
    },
    {
      "action": "downloadPdf",
      "invoice": "{{invoice}}"
    }
  ]
}
```

_With JavaScript:_

When using JavaScript, the result should be an array of objects or values. If the result is a promise, it will be awaited.

```json
{
  "action": "extractAll",
  "script": "Array.from(document.querySelectorAll('#year-selector option')).map(option => option.value);",
  "variable": "year",
  "forEach": [
    {
      "action": "dropdownSelect",
      "selector": "#year-selector",
      "value": "{{year}}"
    }
  ]
}
```

**Pagination**

Experimental support, not yet documented.

#### Extract Network Response (`extractNetworkResponse`)

Extracts data from a network response by matching the URL path. The URL parameter is used as a substring match against network requests, so you only need to provide a unique part of the URL path.

The response content is automatically parsed as JSON if possible, otherwise returned as a string. You can optionally transform the response using the `transform` parameter.

```json
{
  "action": "extractNetworkResponse",
  "url": "api/invoices",
  "variable": "response"
}
```

With transformation:

```json
{
  "action": "extractNetworkResponse",
  "url": "api/data",
  "variable": "items",
  "transform": "(response) => response.items.map(item => ({ id: item.id, date: item.createdAt, total: `${item.currency} ${item.amount}` }))"
}
```

The transformed data will be stored in the specified variable and can be used in subsequent steps.

### 📄 Document Retrieval Steps

These steps are used to download documents and process them in Invoice Radar. All steps require the `document` object to be passed as an argument, which contains the metadata of the document.

The `document` argument has the following fields:

_Required_

- `id`: The unique document ID
  - E.g. `INV-123` or `123456`
- `date`: The invoice date as string
  - E.g. `2022-01-01` or `01/01/2022` or `January 1, 2022`

_Recommended_

- `total`: The invoice total amount including the currency.

  - E.g. `$100.00` or `€100.00` or `100 EUR` or `100,00€`
  - The built in parser will try to extract the amount and currency from the string.

_Optional_

- `type`: The type of the document _(Optional. Defaults to `auto`)_
  - Can be set to `auto`, `invoice`, `outgoing_invoice`, `receipt`, `refund` or `other`.
  - `auto` will try to detect the type based on the document.
- `metadata`: Additional metadata for the document _(Optional)_
  - E.g. `{ "orderNumber": "12345" }`

You can either pass every field separately or the whole object if it contains all required fields.

_E.g. using separate fields:_

```json
"document": {
  "id": "{{item.invoiceId}}",
  "date": "{{item.date}}",
  "total": "{{item.amount}} {{item.currency}}",
  "type": "invoice"
}
```

_E.g. if the object contains all required fields, you can pass it directly:_

```json
"document": "{{item}}"
```

#### Download PDF (`downloadPdf`)

Downloads a PDF from the given URL.

```json
{
  "action": "downloadPdf",
  "url": "https://example.com/invoice.pdf",
  "document": {
    "id": "{{item.invoiceId}}",
    "date": "{{item.date}}",
    "total": "{{item.total}}"
  }
}
```

#### Wait for PDF Download (`waitForPdfDownload`)

Waits for a PDF download. Timeout defaults to 15 seconds.

```json
{
  "action": "waitForPdfDownload",
  "timeout": 10000,
  "document": {
    "id": "{{item.invoiceId}}",
    "date": "{{item.date}}",
    "total": "{{item.total}}"
  }
}
```

#### Print Page as PDF (`printPdf`)

Prints the current page to a PDF file.

```json
{
  "action": "printPdf",
  "document": {
    "id": "{{item.invoiceId}}",
    "date": "{{item.date}}",
    "total": "{{item.total}}"
  }
}
```

#### Download Base64 PDF (`downloadBase64Pdf`)

Downloads a PDF from a base64 encoded string.

```json
{
  "action": "downloadBase64Pdf",
  "base64": "{{item.base64String}}",
  "document": {
    "id": "{{item.invoiceId}}",
    "date": "{{item.date}}",
    "total": "{{item.total}}"
  }
}
```

### 🔀 Conditional Logic Steps

#### If (`if`)

Runs the given steps if the condition is true. If the condition is false, the `else` steps are executed.

```json
{
  "action": "if",
  "script": "'{{invoice.url}}'.includes('pdf')",
  "then": [
    {
      "action": "click",
      "selector": "#example"
    }
  ],
  "else": [
    {
      "action": "navigate",
      "url": "https://example.com/fallback"
    }
  ]
}
```

### 📦 Miscellaneous Steps

#### Sleep (`sleep`)

Waits for the given amount of time in milliseconds.
This is generally not recommended. In most cases, it's better to use the [waitForElement](#wait-for-element-waitforelement), [waitForURL](#wait-for-url-waitforurl) or [waitForNetworkIdle](#wait-for-network-idle-waitfornetworkidle) steps.

```json
{
  "action": "sleep",
  "duration": 1000
}
```

#### Expose Option (`exposeOption`)

Exposes a configuration option to the user. The `config` field is the key of one of the options in the `configSchema`.

For example, if the `configSchema` is `{ "teamId": { "type": "string", "label": "Team ID" } }`, the `config` field can be `teamId`.

```json
{
  "action": "exposeOption",
  "config": "teamId",
  "option": "{{option}}"
}
```

### ✂️ Snippets

Snippets are pre-built sets of steps that simplify common tasks. The steps for a specific snippet are visible inside the developer tools

Currently, it's not possible to create custom snippets. If you have a common task that you think would be useful as a snippet, please create an issue on GitHub.

#### Get Invoice from Stripe URL (`getInvoiceFromStripeUrl`)

Extracts an invoice from a Stripe invoice URL.

```json
{
  "action": "runSnippet",
  "snippet": "getInvoiceFromStripeUrl",
  "args": {
    "url": "https://invoice.stripe.com/i/inv_123"
  }
}
```

#### Get Invoices from Stripe Customer Portal (`getInvoicesFromStripeBillingPortal`)

Extracts available invoices from a Stripe billing portal.

```json
{
  "action": "runSnippet",
  "snippet": "getInvoicesFromStripeBillingPortal",
  "args": {
    "url": "https://stripe-portal.example.com/billing"
  }
}
```

## Advanced Patterns

### Running a fetch request

Sometimes, you might need to run a fetch request inside a step to fetch data from an API. To do this, you can use the `extractAll` action.

```json
{
  "action": "extractAll",
  "variable": "invoice",
  "script": "fetch('https://example.com/api/invoices').then(res => res.json())"
  "forEach": [
    {
      "action": "downloadPdf",
      "url": "{{invoice.url}}",
      "document": {
        "id": "{{invoice.id}}",
        "date": "{{invoice.date}}",
        "total": "{{invoice.total}}"
      }
    }
  ]
}
```

This will run the fetch request and return the result as a JavaScript object.

### Run steps inside an `<iframe/>`

In some scenarios, you might need to run a step inside an `<iframe/>` element. To do this, you can use the `iframe` attribute on the step.

```json
{
  "action": "click",
  "selector": "#button-inside-iframe",
  "iframe": true
},
```

By setting `iframe` to `true`, Invoice Radar will find the first `<iframe/>` element on the page and run the step inside it.

You can also use a string that is contained inside the iframe's `src` attribute to target a specific iframe.

```json
{
  "action": "click",
  "selector": "#button-inside-iframe",
  "iframe": "iframe.example.com"
},
```

## Autofill Configuration

The autofill feature helps streamline the authentication process by automatically filling login credentials. Configure this feature using the top-level `autofill` property in your plugin configuration.

### Default Behavior

By default, `autofill` is set to `true`, which enables automatic input detection for:

- **Username** (also works for email fields)
- **Password**
- **OTP** (One-Time Password/2FA codes)
- **Submit** (automatically submits the form after filling credentials)

When enabled, Invoice Radar will automatically detect and fill these fields during the authentication process, then submit the form to complete the login.

### Customization Options

You can customize or disable autofill for individual fields by setting each property (`username`, `password`, `otp`, `submit`) to either:

- **`false`** - Disables autofill for that specific field
- **Configuration object** - Customizes the behavior with these options:
  - `selector`: Custom CSS selector if the default detection doesn't work
  - `title`: Custom label for the field in the UI (e.g., "Email" instead of "Username")

### Configuration Examples

#### Disable Autofill Completely

```json
{
  "autofill": false
}
```

#### Disabling automatic submission

```json
{
  "autofill": {
    "submit": false
  }
}
```

#### Custom Configuration

```json
{
  "autofill": {
    "username": {
      "selector": "#email-input",
      "title": "Email Address"
    },
    "password": false,
    "submit": {
      "selector": "#login-submit"
    }
  }
}
```
