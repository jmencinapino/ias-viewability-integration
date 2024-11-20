# IAS Viewability Integration

This repository provides a guide for integrating an IAS Viewability Pixel.

>[!NOTE]
> This product is not officially supported. The following information is part of a workaround provided by Equativ's team. Each integration case could require individual treatment based on unique requirements.
---
## Context

To measure viewability, IAS typically provides three main items:

1. A third-party pixel (e.g. DoubleClick, etc).
2. A unique IAS library reference, usually skeleton.js.
3. A no-script pixel reference is used as a backup for the main library, usually skeleton.gif.

Equativ's Customized Script will integrate these three elements at the insertion level.

---

## Integration

### Ad Container Reference

During configuration, IAS requires referencing the container that requires measurement.

This container must be identified using either the ID or Class selector.

ID: using %23.
```
E.g. %23IDName
```

Class: using (.).
```
E.g. .className
```
> [!IMPORTANT]
> `%23` is the encoded version of the character `#`.

---

### How Can I Determine What is the Proper Selector?

Review the ad container where creatives will be displayed.

By default, if there are no custom references, the ad container reference will be `sas_formatId`.

---
 
### Script Integration

To simplify the integration of the code into the Custom Script at the insertion level, we developed the code below.

As a client, it is needed to modify the IAS Configuration Parameters by adding your own IAS references, following the standard information provided by IAS as mentioned above.

- iasPixel: URL
- iasLibrary: URL
- backupPixel: URL
- adContainerRef: ID/Class selector

> [!NOTE]
> This field is using `%23[sas_tagId]` by default. `[sas_tagId]` corresponds to a macro in Equativ which will be replaced by the value: `sas_formatId`.

---

Code example:
```javascript
let iasConfig = {
    iasPixel: 'https://ad.doubleclick.net/track/pixel-URL',
    iasLibrary: 'https://pixel.com/123456/7891/skeleton.js',
    backupPixel: 'https://pixel.com/123456/7891/skeleton.gif',
    /* %23 -> ID selector || . -> Class selector */
    adContainerRef: '%23[sas_tagId]'
};
```

The rest of the present code will create an element for each of the available URLs.

Code does not need to be modified for standard/default banner integrations.

```javascript
(function(w){
    // IAS Configuration Parameters
    let iasConfig = {
        iasPixel: 'https://ad.doubleclick.net/ddm/trackimp/pixel-URL',
        iasLibrary: 'https://pixel.adsafeprotected.com/rjss/st/123456/7891/skeleton.js',
        backupPixel: 'https://pixel.adsafeprotected.com/rfw/st/123456/7891/skeleton.gif?gdpr=[sas_gdpr_applies]&gdpr_consent=[sas_gdpr_consent]&gdpr_pd=',
        adContainerRef: '%23[sas_tagId]' // %23 -> ID selector || . -> Class selector  
    };
    // Basic DOM Elements
    var d = w.document, h = d.getElementsByTagName('head')[0], b = d.body;
    
    // IAS extra container (if needed)

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
    // IAS Skeleton Backup Pixel declaration
    var backupCode = `<IMG SRC="${iasConfig.backupPixel}" BORDER=0 WIDTH=1 HEIGHT=1 ALT="">`,
    backupPixel = d.createElement('NOSCRIPT');
    backupPixel.type ='application/javascript';
    backupPixel.appendChild(d.createTextNode(backupCode));

    // IAS Code Injection
    if (typeof w.sas_ajax != 'undefined' && w.sas_ajax) {
        if(extraContainer) w.sas_appendToContainer([sas_formatId] , extraContainer);
        w.sas_appendToContainer([sas_formatId] , iasPixel);
        w.sas_appendToContainer([sas_formatId] , iasLibrary);
        w.sas_appendToContainer([sas_formatId] , backupPixel);
    }
    else {
        if(extraContainer)b.appendChild(extraContainer);
        b.appendChild(iasPixel);
        b.appendChild(iasLibrary);
        b.appendChild(backupPixel);
    }
})(window.top);
```
> [!NOTE]
> See also the complete version of the code at `ias-custom-script-standard.html` file attached to this repository.

---

## Special Cases: Background Integration

When using this integration for non-standard ad formats, such as a Background template, IAS typically requires the implementation of an additional container. 

The additional container must:
- Adhere to a minimum dimension of 50x50 pixels
- Contain a transparent Image as src
- Remain visible during user interaction in accordance with IAS requirements.
 
This is a practical example of the image container that must be integrated at the specified position (IAS extra container).
```javascript
// IAS extra container (if needed)
extraContainer = d.createElement('IMG');
extraContainer.setAttribute('id','ias-counting');
extraContainer.setAttribute('src','https://domain.com/transparent.png');
extraContainer.setAttribute('style','width:100px;height:100vh;border:0px;display:block;position:absolute;top:20px;');
extraContainer.appendChild(document.createTextNode(code));
```

For this particular scenario, IAS Configuration Parameters should be also updated, following that adContainerRef should be replaced by: `%23ias-counting`.

```javascript
// IAS Configuration Parameters
let iasConfig = {
    // rest of parameters
    adContainerRef: '%23ias-counting'
};
```
> [!NOTE]
> See the complete code of this use case at `ias-custom-script-background.html` file attached to this repository.
---

## Disclaimer

While this is used for most IAS viewability interactions, we recommend that you consult your IAS contact if it does not work as expected to improve and optimize this integration.

This is a dedicated custom integration created specifically for IAS integration.
