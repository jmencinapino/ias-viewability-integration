# IAS Viewability Integration

This repository provides a guide for integrating an IAS Viewability Pixel.

>[!NOTE]  
>This product is not officially supported. The following information is part of a workaround provided by Equativ's service teams. Each integration case may require individual treatment based on unique requirements.

---

## Context

Integral Ad Science (IAS) provides clients with the ability to measure both impressions and viewability through the integration of several scripts and elements.  
These elements must be integrated at Equativ's insertion/creative level to ensure accurate measurement.

## Overview

To proceed with the integration, IAS typically provides three main items:

1. A third-party pixel (e.g. DoubleClick).
2. A unique IAS library reference, usually `skeleton.js`.
3. A no-script pixel reference used as a backup, usually `skeleton.gif`.

Here is an example of the code originally provided by IAS:
```html
<!-- Third-part pixel -->
<img src="https://ad.doubleclick.net/ddm/trackimp/pixel-URL" alt="" width="0" height="0" border="0">
<!-- IAS Library / skeleton.js -->
<script src="https://pixel.adsafeprotected.com/rjss/st/123456/7891/skeleton.js"></script>
<!-- NoScript pixel -->
 <noscript><img src="https://pixel.adsafeprotected.com/rfw/st/123456/7891/skeleton.gif?gdpr=[sas_gdpr_applies]&gdpr_consent=[sas_gdpr_consent]&gdpr_pd=" alt=""></noscript> 
```

Equativ's Customized Script feature provides a place where this code can be integrated, following the steps described below.

---

# Integration

## 1. Ad Container Reference

During configuration, IAS requires referencing the container that needs measurement.

Determine what kind of container reference will be required by reviewing the ad container where creatives will be displayed. By default, the ad container reference will be the ID `sas_formatId` (also included as a macro like `[sas_tagId]`).

IAS will use the container's information to reference the proper container that needs to be measured with:

- ID by using the character `%23` (encoded `#`). E.g., `%23id_ref`
- Class by using the character `.`. E.g., `.class_ref`

---

## 2. Main Integration

### Script Library Integration

The following code was created to simplify the integration of the code into the Custom Script at the insertion level.

As a client, you need to modify the IAS Configuration Parameters by adding your own IAS references, providing values for the following parameters:

- `iasPixel`: URL
- `iasLibrary`: URL
- `adContainerRef`: ID/Class selector

This means clients only need to use URL references as part of the configuration.

#### Code example:
```javascript
let iasConfig = {
    iasPixel: 'https://ad.doubleclick.net/track/pixel-URL',
    iasLibrary: 'https://pixel.com/123456/7891/skeleton.js',
    /* %23 -> ID selector || . -> Class selector */
    adContainerRef: '%23[sas_tagId]'
};
```

#### NoScript Integration

In addition IAS provides a `noscript` snippet for integrating a Skeleton Backup Pixel.

This portion of the code should be kept as provided.

```html
<noscript><img src="SKELETON_BCKUP_PIXEL_URL" alt=""></noscript>
```

#### Complete Code
The example below illustrates how the final integration of the IAS code should be structured.

```html
<!-- IAS CUSTOM SCRIPT STANDARD SNIPPET -->
<script type="application/javascript">
    (function(w){
        // IAS Configuration Parameters
        let iasConfig = {
            iasPixel: 'https://ad.doubleclick.net/ddm/trackimp/pixel-URL',
            iasLibrary: 'https://pixel.adsafeprotected.com/rjss/st/123456/7891/skeleton.js',
            adContainerRef: '%23[sas_tagId]' // %23 -> ID selector || . -> Class selector  
        };
        // Basic DOM Elements
        var d = w.document, h = d.getElementsByTagName('head')[0], b = d.body;

        // IAS Pixel declaration
        iasPixel = d.createElement('IMG');
        iasPixel.type ='image';
        iasPixel.setAttribute('style', 'width:1px;height:1px;border:0px;');
        iasPixel.setAttribute('src', iasConfig.iasPixel);

        // IAS Skeleton Library declaration
        iasLibrary = d.createElement('SCRIPT');
        iasLibrary.type = 'application/javascript';
        iasLibrary.setAttribute('async', 'true');
        iasLibrary.setAttribute('src', iasConfig.iasLibrary + '?ias_adpath=' + iasConfig.adContainerRef);
 
        // IAS Code Injection
        if (typeof w.sas_ajax != 'undefined' && w.sas_ajax) {
            if(extraContainer) w.sas_appendToContainer([sas_formatId] , extraContainer);
            w.sas_appendToContainer([sas_formatId] , iasPixel);
            w.sas_appendToContainer([sas_formatId] , iasLibrary);
        }
        else {
            if(extraContainer)b.appendChild(extraContainer);
            b.appendChild(iasPixel);
            b.appendChild(iasLibrary);
        }
    })(window.top);
</script>

<!-- IAS Skeleton Backup Pixel -->
<noscript>
    <img src="https://pixel.adsafeprotected.com/rfw/st/123456/7891/skeleton.gif?gdpr=[sas_gdpr_applies]&gdpr_consent=[sas_gdpr_consent]&gdpr_pd=" alt="">
</noscript>
```
> [!NOTE]
> See also the complete version of the code at `ias-custom-script-standard.html` file attached to this repository.

---

## 3. Specific Scenarios
Equativ does not provide a default container reference in the following scenarios; therefore, this configuration must consider several details.

### Interstitial Ad Integration

Interstitial Ads are not displayed directly into Equativ's default container given the display is required to take into account the whole screen size.

In this particular case, the container id that needs to be referenced is: `sas-interstitial`.

The code should include the change on the `adContainerRef` parameter.

```javascript
let iasConfig = {
    // (...)
    adContainerRef: '%23sas-interstitial'
};
```
### Background Ad Integration

When Background Ads are displayed, IAS requires the implementation of an additional container described as:
- Size: 50x50px (min.).
- Implement a transparent image as src.
- Remain visible during user interaction.
 
Practical example IAS extra container:
```javascript
// IAS extra container (if needed)
extraContainer = d.createElement('IMG');
extraContainer.setAttribute('id','ias-counting');
extraContainer.setAttribute('src','https://domain.com/transparent.png');
extraContainer.setAttribute('style','width:100px;height:100vh;border:0px;display:block;position:absolute;top:20px;');
extraContainer.appendChild(document.createTextNode(code));
```

As for Interstitial Ads, the adContainerRef needs to be filled with the new container id: `%23ias-counting`.
```javascript
// IAS Configuration Parameters
let iasConfig = {
    // (...)
    adContainerRef: '%23ias-counting'
};
```
> [!NOTE]
> See the complete code of this use case at `ias-custom-script-background.html` file attached to this repository.
---

## Disclaimer

> [!IMPORTANT]  
> Third-party integrations must be tested in a controlled environment before using them as part of the Equativ's creative configuration. It is recommended to contact with IAS if it does not work as expected to improve and optimize this integration.
> Ensure compliance with privacy regulations (e.g., GDPR, CCPA) when handling user data, and confirm that the integration aligns with the [official Custom Script feature documentation](https://help.smartadserver.com/s/article/Configuring-creatives#:~:text=unchecked%20by%20default.-,Custom%20script%20(for%20creatives),-You%20can%20add).

> [!WARNING]  
> External scripts can introduce security risks or performance degradation if not implemented properly.
> Equativ is only responsible for executing the Custom Script feature. Any actions performed during the script execution itself are not the responsibility of Equativ.