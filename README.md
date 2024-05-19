# InvoiceRadar Plugin Handbook

## 1. Introduction

Welcome to the InvoiceRadar Plugin Handbook for developers! This guide will help you create custom plugins to fetch invoices and receipts from various platforms.

InvoiceRadar is a document automation tool that helps you fetch, download, and organize invoices and receipts from various platforms.

[📟 Learn more about InvoiceRadar](https://invoiceradar.com/)

#### Table of Contents

1. [Introduction](#1-introduction)
2. [Getting Started](#2-getting-started)
3. [Plugin Structure](#3-plugin-structure)
4. [Writing Your First Plugin](#4-writing-your-first-plugin)
5. [Useful Patterns](#5-useful-patterns)
6. [Steps Reference](#steps-reference)

## 2. Getting Started

### Prerequisites

- Basic knowledge of JSON, HTML, CSS and JavaScript.
- A text editor or IDE (e.g., VSCode, Sublime Text).
- InvoiceRadar installed on macOS or Windows.

### Installation

1. **Download and Install InvoiceRadar**:

   - [Request Access to InvoiceRadar](https://invoiceradar.com/)

2. **Download the Blank Plugin**:

   - [Download the Blank Plugin](https://github.com/invoiceradar/plugins/blob/main/blank-plugin.json) to your local machine.
   - Rename the file to `your-plugin-name.json`.
   - Put it into a folder of your choice.

3. **Add the Plugin to InvoiceRadar**:
   - Open InvoiceRadar.
   - Navigate to settings and choose the `Available Plugins` tab.
   - Choose `Choose Plugin Directory` and select the folder where you saved the plugin.
   - Your plugin should now appear in the list of available plugins.

## 3. Plugin Structure

Plugins for InvoiceRadar are written in JSON and follow a specific structure. Each plugin consists of the following sections:

**Plugin Description**:

- **Metadata**: Basic information about the plugin, such as name, description, and homepage URL.
- **configSchema**: Configuration properties that the plugin may require.

**Scraping Steps**:

- **checkAuth**: Steps to verify if the user is already authenticated.
- **startAuth**: Steps to initiate the authentication process.
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

   This information is used to identify and display the plugin in InvoiceRadar. The homepage URL is used to get the favicon of the service.

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

   These fields will be displayed to the user when adding the plugin in InvoiceRadar.

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

5. **Scrape Documents**:

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
             "jonas": "{{invoice.date}}",
             "date": "{{invoice.date}}",
             "total": "{{invoice.total}}"
           }
         }
       ]
     }
   ]
   ```

6. **You are done!**:

   Save the file and add it to InvoiceRadar. You can now run the plugin to fetch invoices from the service.

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
  "url": "https://example.com/dashboard",
}
```

Depending on the service, you might do it the other way around.

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

#### Pattern 2: Check for Logout Button

You can use a selector that is unique to the authenticated state to check if the user is authenticated, e.g. a logout button or profile link.

```json
{
  "action": "navigate",
  "url": "https://example.com/login"
},
{
  "action": "waitForElement",
  "selector": "#logout-button"
}
```

#### Tip: Make sure the website is fully loaded

In some cases, the website has not fully loaded when the `checkElementExists` step is executed. To avoid this, you can use the `waitForElement` step to wait for a specific element to appear on the page.

```json
{
  "action": "navigate",
  "url": "https://example.com/dashboard"
},
{
  "action": "waitForElement",
  "selector": "#main-app"
},
{
  "action": "checkElementExists",
  "selector": "#logout-button"
}
```

Note that the selector `#main-app` has to be available in both states (logged in and logged out). Otherwise, the step will timeout.

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

This section provides an overview of the available steps that can be used to create plugins for InvoiceRadar. Each step represents a specific action that can be performed during the automation process.

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

- [📄 Document Retrieval Steps](#📄-document-retrieval-steps)

  - [Download PDF (`downloadPdf`)](#download-pdf-downloadpdf)
  - [Wait for PDF Download (`waitForPdfDownload`)](#wait-for-pdf-download-waitforpdfdownload)
  - [Print Page as PDF (`printPdf`)](#print-page-as-pdf-printpdf)
  - [Get Invoice from Stripe URL (`getInvoiceFromStripeUrl`)](#get-invoice-from-stripe-url-getinvoicefromstripeurl)

- [🔀 Conditional Logic Steps](#🔀-conditional-logic-steps)

  - [If (`if`)](#if-if)

- [📦 Miscellaneous Steps](#📦-miscellaneous-steps)
  - [Sleep (`sleep`)](#sleep-sleep)

### 🌐 Navigation Steps

#### Navigate (`navigate`)

Navigates to the given URL and waits for the page to load. Note that this only waits for the initial page load, not for any subsequent AJAX requests. You can [waitForNetworkIdle](#wait-for-network-idle-waitfornetworkidle) if you need to wait for all resources to be loaded.

```json
{
  "action": "navigate",
  "url": "https://example.com"
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
  "url": "https://example.com/profile/*",
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

Waits for the page to finish navigating. Timeout is optional and defaults to 15 seconds.

```json
{
  "action": "waitForNavigation",
  "timeout": 10000
}
```

#### Wait for Network Idle (`waitForNetworkIdle`)

Waits for the network to be idle. This is useful if you want to ensure the page has finished loading all resources. The steps completes when there are no more network requests for 1000ms. Timeout is optional and defaults to 15 seconds.

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

Checks if the current URL matches the given URL. Supports wildcards.

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
  - Can be set to `auto`, `invoice`, `receipt`, `refund` or `other`.
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

#### Get Invoice from Stripe URL (`getInvoiceFromStripeUrl`)

Extracts the invoice PDF and details from a Stripe invoice URL.

```json
{
  "action": "getInvoiceFromStripeUrl",
  "url": "https://invoice.stripe.com/i/inv_123"
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
This is generally not recommended. In most cases, it's better to use the [waitForElement](#wait-for-element-waitforelement) or [waitForURL](#wait-for-url-waitforurl).

```json
{
  "action": "sleep",
  "duration": 1000
}
```
