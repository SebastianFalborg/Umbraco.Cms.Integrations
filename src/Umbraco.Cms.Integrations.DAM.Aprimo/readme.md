# Umbraco.Cms.Integrations.DAM.Aprimo

This integration provides a custom media picker for digital assets managed in an Aprimo workspace. It can be used as a property editor for content with a value converter providing a strongly typed model for rendering, as well as a sample rendering component.

## Prerequisites

Requires minimum versions of Umbraco CMS: 
- CMS: 10.3.0

## How To Use

### Configuration

The following configuration is required to connect to the _Aprimo_ DAM workspace:

```
{
  "Umbraco": {
    "CMS": {
      "Integrations": {
        "DAM": {
          "Aprimo": {
            "Settings": {
              "Tenant": "[your_tenant_name]"
            },
            "OAuthSettings": {
              "ClientId": "[your_aprimo_client_id]",
              "ClientSecret": "[your_aprimo_client_secret]",
              "RedirectUri": "https://[your_website_base_url]/umbraco/api/aprimoauthorization/oauth",
              "Scopes": "api offline_access"
            }
          }
        }
      }
    }
  }
}
```

The configuration is split in two components: a generic one that holds your tenant name, and one for OAuth settings.

The authorization process is managed using the [OAuth flow - Authorization Code with PKCE](https://developers.aprimo.com/marketing-operations/rest-api/authorization/#module7), meaning that when making a request for an access token, you will need to generate a code verifier and a code challenge. 

`Client Id` and `Client Secret` details are retrieved from your [Aprimo Client Registration](https://developers.aprimo.com/marketing-operations/rest-api/authorization/#module2), which you will need to set up with the redirect URI pointing to the [`AprimoAuthorizationController`](https://github.com/umbraco/Umbraco.Cms.Integrations/blob/feature/aprimo-integration/src/Umbraco.Cms.Integrations.DAM.Aprimo/Controllers/AprimoAuthorizationController.cs) - `OAuth` action.

Using the `offline_access` scope, the response from Aprimo API will contain besides an access token, a refresh token, with an expiration period of 10 days, that will keep the API access valid.

### Backoffice usage
To use the media picker, a new data type should be created based on the `Aprimo Media Picker` property editor.

If the configuration is not valid, an error tag will be displayed in the right upper corner of the configuration box.

Otherwise, you will be able to select one of the two available options for picking assets:
- Aprimo API - items are retrieved using the API and an overlay of the property editor will display the list of available items in the DAM workspace.
- Aprimo Content Selector - rich UI tool hosted on Aprimo Cloud where you can pick items using a familiar Aprimo interface.

#### Browser information
Before using the integration with Aprimo, please make sure to use a browser that is supported by Aprimo Cloud, in contrary you will not be able to authenticate, nor use the Content Selector.

Aprimo currently supports these browsers, but make sure to check [this](https://help.aprimo.com/Content/Marketing_Operations_Help/aprimo_basics/browsers_configuring_concept.html) topic for an updated list:
- Chrome for Windows and Macintosh
- Edge (Windows 10 only)
- Internet Explorer 9, 10, and 11
- Safari 6, 6.1, 6.2, and 7.0 (Macintosh only)

### Front-end rendering
A strongly-typed model will be generated by the property value converter and an HTML helper is available to easily render the asset on the front-end.

Make sure your template has a reference to the following using statement:
`@using Umbraco.Cms.Integrations.DAM.Aprimo.Helpers;`

And render the media asset (assuming a property based on the created data type with alias `AprimoMediaPicker` has been created):
`@Html.RenderAprimoAsset(Model.AprimoMediaPicker)`.

Properties available from the strongly-typed model:
- Title
- Thumbnail
- Crops
- Asset fields

### Working with Crops
For the selected media asset you can retrieve the crops details using the `MediaWithCrops` object.
 
It contains the details of the original asset, the list of available crops and a method to retrieve the crop URL based on name and width/height.

For example:
- get URL for crop item with the name _Social_: `@Model.MediaWithCrops.GetCropUrl("Social")`
- get URL for crop item with height _1080_: `@Model.MediaWithCrops.GetCropUrl(null, 1080)`

### Working with fields
The asset's fields are grouped in an object containing their label and a dictionary of values based on the available cultures for that asset.

For example:
- get values for a field with label _Display Title_: 
```
var displayTitle = @Model.Fields.FirstOrDefault(p => p.Label == "Display Title");
var values = displayTitle != null
                ? displayTitle.Values 
                : default(Dictionary<string, string>());
``` 

### Version history
- 1.0.0 - Initial release