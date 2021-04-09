# Background
The WebView2 team has been asked for an API to allow end developers to clear the browsing data that is stored in the User Data Folder. Developers want to be able to clear data between each of their customers, clear the data folder to clear space, and to clear data on the fly. 
Currently users can delete the entire User Data Folder to clear this data. This has a few drawbacks: it removes the entire user data folder instead of specific parts which incurs performance costs later on, the webview must be shutdown fully and then re-initialized, and deleting the User Data Folder is a complex API to call. 
We are creating an API that will allow developers to clear the browsing data programtically in which the developer can specify the data kind to clear. 

In this document we describe the updated API. We'd appreciate your feedback.


# Description
The clear browsing data API is an asynchronous API. 
The browsing data kinds that are supported are listed below.  These data kinds follow a hierarchical structure in which nested bullet points are included in their parent bullet point's data kind. 

Ex: Dom storage is encompassed in site data which is encompassed in the profile data. Each of the following bullets correspond to a COREWEBVIEW2_BROWSING_DATA_KIND unless otherwise specified. 

* Profile 
    * Site Data
        * Dom Storage 
            * App Cache
            * File Systems
            * Indexddb
            * Local Storage
            * Web SQL
            * Cache Storage
            * Embedder Dom Storage (this API does not support this specific data kind but this is included in the dom storage data kind)
            * Background Fetch (this API does not support this specific data kind but this is included in the dom storage data kind)
        * Cookies
        * Media Licenses
        * Plugin Data
        * Site Usage
        * Durable Permissions
        * External Protocols
        * Isolated Origins
        * Trust Tokens
        * Conversions Data
    * Http Cache 
    * Download History
    * General Autofill
    * Password Autofill
    * Bookmarks
    * Settings
    * Content Settings
    * Local Custom Dictionary


# Examples

## Win32 C++
```cpp
/// Function to clear the password form data 
bool EnvironmentComponent::ClearPasswordAutofillData() 
{
    wil::com_ptr<ICoreWebView2Environment5> environment;
    webView->get_Environment(&environment);
    CHECK_FAILURE(environment->ClearBrowsingData(COREWEBVIEW2_BROWSING_DATA_KIND_PASSWORD_AUTOFILL,
        Callback<ICoreWebView2ClearBrowsingDataHandler>(
            [this](HRESULT error, bool success) -> HRESULT {
                if (success){
                    return true;
                } else {
                    return false;
                }
                return S_OK;
            })
            .Get()));
}
```

### .NET, WinRT
```c#
webView2Control.NavigationStarting += ClearPasswordAutofillData;

private void ClearPasswordAutofillData() 
{
    var environment = webView2Control.CoreWebView2.Environment;
    try
    {
        await environment.ClearBrowsingDataAsync(COREWEBVIEW2_BROWSING_DATA_KIND_PASSWORD_AUTOFILL); 
    }
    catch (System.Runtime.InteropServices.COMException exception)
    {
        // An exception occured
    }
}

```

# API Notes
See [API Details](#api-details) section below for API reference.


# API Details

## Win32 C++

```IDL
interface ICoreWebView2Environment5
interface ICoreWebView2ClearBrowsingDataCompletedHandler;

/// Specifies the datatype for the 
/// `ICoreWebView2Environment5::ClearBrowsingData` method.
[v1_enum]
typedef enum COREWEBVIEW2_BROWSING_DATA_KIND {
  /// Specifies app cache data.
  COREWEBVIEW2_BROWSING_DATA_KIND_APP_CACHE = 0X01,

  /// Specifies file systems data.
  COREWEBVIEW2_BROWSING_DATA_KIND_FILE_SYSTEMS = 0X02,

  /// Specifies indexeddb data.
  COREWEBVIEW2_BROWSING_DATA_KIND_INDEXEDDB = 0X03,

  /// Specifies local storage data.
  COREWEBVIEW2_BROWSING_DATA_KIND_LOCAL_STORAGE = 0X04,

  /// Specifies web SQL data.
  COREWEBVIEW2_BROWSING_DATA_KIND_WEB_SQL = 0X05,

  /// Specifies cache storage.
  COREWEBVIEW2_BROWSING_DATA_KIND_CACHE_STORAGE = 0X06,

  /// Specifies dom storage data. This browsing data kind is inclusive 
  /// of COREWEBVIEW2_BROWSING_DATA_KIND_APP_CACHE, COREWEBVIEW2_BROWSING_DATA_KIND_FILE_SYSTEMS,
  /// COREWEBVIEW2_BROWSING_DATA_KIND_INDEXEDDB, COREWEBVIEW2_BROWSING_DATA_KIND_LOCAL_STORAGE,
  /// COREWEBVIEW2_BROWSING_DATA_KIND_WEB_SQL, COREWEBVIEW2_BROWSING_DATA_KIND_CACHE_STORAGE.
  /// In addition to these data kinds, dom storage also includes embedder dom storage and
  /// background fetch. 
  COREWEBVIEW2_BROWSING_DATA_KIND_DOM_STORAGE = 0x07,
  
  /// Specifies http cookies data.
  COREWEBVIEW2_BROWSING_DATA_KIND_COOKIES = 0x08,

  /// Specifies media licenses.
  COREWEBVIEW2_BROWSING_DATA_KIND_MEDIA_LICENSES = 0X09,
  
  /// Specifies plugin data.
  COREWEBVIEW2_BROWSING_DATA_KIND_PLUGIN = 0X10,

  /// Specifies site usage data.
  COREWEBVIEW2_BROWSING_DATA_KIND_SITE_USAGE = 0X11,

  /// Specifies durable permissions data. 
  COREWEBVIEW2_BROWSING_DATA_KIND_DURABLE_PERMISSIONS = 0X12,

  /// Specifies external protocols data.
  COREWEBVIEW2_BROWSING_DATA_KIND_EXTERNAL_PROTOCOLS = 0X013,

  /// Specifies isolated origins data.
  COREWEBVIEW2_BROWSING_DATA_KIND_ISOLATED_ORIGINS = 0X14,

  /// Specifies trust tokens data.
  COREWEBVIEW2_BROWSING_DATA_KIND_TRUST_TOKENS = 0X15,

  /// Specifies conversions data (the completion of meaningful user action on 
  /// the advertiser's web site by a user who has interacted with an ad from that advertiser).
  COREWEBVIEW2_BROWSING_DATA_KIND_CONVERSIONS = 0X16,

  /// Specifies site data. This browsing data kind 
  /// is inclusive of COREWEBVIEW2_BROWSING_DATA_KIND_DOM_STORAGE,
  /// COREWEBVIEW2_BROWSING_DATA_KIND_COOKIES, COREWEBVIEW2_BROWSING_DATA_KIND_MEDIA_LICENSES,
  /// COREWEBVIEW2_BROWSING_DATA_KIND_PLUGIN, COREWEBVIEW2_BROWSING_DATA_KIND_SITE_USAGE,
  /// COREWEBVIEW2_BROWSING_DATA_KIND_DURABLE_PERMISSIONS, COREWEBVIEW2_BROWSING_DATA_KIND_EXTERNAL_PROTOCOLS,
  /// COREWEBVIEW2_BROWSING_DATA_KIND_ISOLATED_ORIGINS, COREWEBVIEW2_BROWSING_DATA_KIND_TRUST_TOKENS, and
  /// COREWEBVIEW2_BROWSING_DATA_KIND_CONVERSIONS. If on an android OS, site data also includes web app data. 
  COREWEBVIEW2_BROWSING_DATA_KIND_SITE = 0x17, 

  /// Specifies content in the HTTP cache including images and other files. 
  COREWEBVIEW2_BROWSING_DATA_KIND_HTTP_CACHE = 0x18,

  /// Specifies download history data. 
  COREWEBVIEW2_BROWSING_DATA_KIND_DOWNLOAD_HISTORY = 0x19, 

  /// Specifies general autofill form data. 
  /// This excludes password forms and includes information like: 
  /// names, street and email addresses, phone numbers, and arbitrary input. 
  COREWEBVIEW2_BROWSING_DATA_KIND_GENERAL_AUTOFILL = 0x20, 

  /// Specifies password autofill forms data. 
  COREWEBVIEW2_BROWSING_DATA_KIND_PASSWORD_AUTOFILL = 0x21,

  /// Specifies browsing history data. 
  COREWEBVIEW2_BROWSING_DATA_KIND_BROWSING_HISTORY = 0x22,

  /// Specifies bookmarks data.  
  COREWEBVIEW2_BROWSING_DATA_KIND_BOOKMARKS = 0x23,

  /// Specifies settings data.
  COREWEBVIEW2_BROWSING_DATA_KIND_SETTINGS = 0x24,

  /// Specifies content settings data.
  COREWEBVIEW2_BROWSING_DATA_KIND_CONTENT_SETTINGS = 0x25,

  /// Specifies local custom dictionary data. 
  COREWEBVIEW2_BROWSING_DATA_KIND_LOCAL_CUSTOM_DICTIONARY = 0x26,

  /// Specifies profile data that should be wiped to make it look like a new profile. 
  /// This does not delete account-scoped data like passwords but will remove access
  /// to account-scoped data by signing the user out. 
  /// This browsing data kind if inclusive of COREWEBVIEW2_BROWSING_DATA_KIND_SITE,
  /// COREWEBVIEW2_BROWSING_DATA_KIND_HTTP_CACHE, COREWEBVIEW2_BROWSING_DATA_KIND_DOWNLOAD_HISTORY,
  /// COREWEBVIEW2_BROWSING_DATA_KIND_GENERAL_AUTOFILL, COREWEBVIEW2_BROWSING_DATA_KIND_PASSWORD_AUTOFILL,
  /// COREWEBVIEW2_BROWSING_DATA_KIND_BROWSING_HISTORY, COREWEBVIEW2_BROWSING_DATA_KIND_CONTENT_SETTINGS,
  /// COREWEBVIEW2_BROWSING_DATA_KIND_BOOKMARKS, COREWEBVIEW2_BROWSING_DATA_KIND_SETTINGS,
  /// COREWEBVIEW2_BROWSING_DATA_KIND_LOCAL_CUSTOM_DICTIONARY.
  COREWEBVIEW2_BROWSING_DATA_KIND_PROFILE = 0X27,
} COREWEBVIEW2_BROWSING_DATA_KIND;

[uuid(4ecfcb16-dd09-4464-9a71-fd8e2d3ac0a2), object, pointer_default(unique)]
interface ICoreWebView2SEnvironment5 : ICoreWebView2Environment {
  /// Clear browsing data based on a specific data type. The completed handler
  /// will be invoked when the browsing data has been cleared and will indicate 
  /// if the specified data was properly cleared. 
  HRESULT ClearBrowsingData(
    [in] COREWEBVIEW2_BROWSING_DATA_KIND dataKind,
    [in] ICoreWebView2ClearBrowsingDataCompletedHandler *handler);
}

[uuid(c2b78e49-5bf5-4d38-a535-668a8a8a30d9), object, pointer_default(unique)]
interface ICoreWebView2ClearBrowsingDataCompletedHandler : IUnknown {
  /// Provide the completion status and result of the corresponding
  /// asynchronous method.
  HRESULT Invoke([in] HRESULT errorCode, [in] BOOL isSuccessful);
}
```
### .NET, WinRT
```c#
namespace Microsoft.Web.WebView2.Core
{
    public partial class CoreWebView2Environment
    {
        public async Task<bool> ClearBrowsingDataAsync(CoreWebView2BrowsingDataKind dataKind);
    }
}
```