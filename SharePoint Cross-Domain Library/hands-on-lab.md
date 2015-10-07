SharePoint Cross-Domain Library
===
## Overview
Tato uk�zka vych�z� z ofici�ln�ho Office Dev Campu a rozv�j� lab v modulu 3.

C�lem je vytvo�it **provider-hosted** aplikaci pro SharePoint, kter� vyu��v� ASP.NET MVC a implementuje ob� situace spojen� s ```SP.RequestExecutor```:

* vol�n� App Webu
* vol�n� Host Webu

## Cross-Domain Library & App Web
### Vytvo�en� nov� aplikace ve Visual Studiu 2013
...

## Cross-Domain Library & Host Web

_TODO: vytvo�en� seznamu Auta_

Zaj�mav�j�� situace nast�v� v p��pad�, �e chceme p�istupovat na Host Web, tedy mimo na�i vlastn� aplikaci.

1. Spus�te **Visual Studio 2013** jako Administrator.
1. Vyberte **File -> New -> Project**.
1. Zvolte **Templates -> Visual C# -> Office -> SharePoint -> Apps -> App for SharePoint**.
1. **Pojmenujte** projekt podle sv� libosti a klikn�te na **OK**.
1. Zadejte URL sv�ho developer tenantu, nap��klad **https://martinovo.sharepoint.com**.
1. Vyberte **Provider-hosted** a klikn�te na **Next**.<br />
![](Images/new-app.png)
1. Vyberte **SharePoint Online** a klikn�te na **Next**.<br />
![](Images/target.png) 
1. Vyberte **ASP.NET MVC Web Application** a klikn�te na **Next**.
1. Nechte vybran� **Use Windows Azure Access Control Service** a klikn�te na **Finish**.

Vytvo�� se nov� aplikace p�ipraven� pro nasazen� na SharePoint Online.

Nyn� ji za�neme upravovat pro vol�n� pomoc� cross-domain library.

1. Otev�ete soubor **HomeController.cs** ve slo�ce **Controllers**.
1. Sma�te obsah bloku **using** v metod� **Index** a soubor ulo�te.
![](Images/index-method.png)
1. Klikn�te prav�m tla��tkem na slo�ku **Scripts** a zvolte **Add -> JavaScript File**.
1. Pojmenujte soubor **crossdomain.js** (p��padn� jakkoliv jinak).
1. Vlo�te do n�j n�sleduj�c� k�d:<br />
```javascript
(function () {
        "use strict";

        jQuery(function () {

            var appWebUrl = "";
            var spHostUrl = "";
            var args = window.location.search.substring(1).split("&");

            for (var i = 0; i < args.length; i++) {
                var n = args[i].split("=");
                if (n[0] == "SPHostUrl")
                    spHostUrl = decodeURIComponent(n[1]);
            }

            for (var i = 0; i < args.length; i++) {
                var n = args[i].split("=");
                if (n[0] == "SPAppWebUrl")
                    appWebUrl = decodeURIComponent(n[1]);
            }

            var scriptbase = spHostUrl + "/_layouts/15/";

            jQuery.getScript(scriptbase + "SP.RequestExecutor.js", function (data) {

                //Call Host Web with REST
                var executor = new SP.RequestExecutor(appWebUrl);
                executor.executeAsync({
                    url: appWebUrl + "/_api/SP.AppContextSite(@hostweb)/web/lists/getbytitle('Auta')/items?@hostweb='" + spHostUrl +"'",
                    method: "GET",
                    headers: { "accept": "application/json;odata=verbose" },
                    success: function (data) {

                        var results = JSON.parse(data.body).d.results;
                        for (var i = 0; i < results.length; i++) {
                            $("#carList").append("<li>" + results[i].Title + "</li>");
                        }
                    },
                    error: function () {
                        alert("Error!");
                    }
                });

            });

        });

    }());
```
1. Nyn� otev�ete view **Index.cshtml** ve slo�ce **Views -> Home** a vlo�te do n�j n�sleduj�c�:

```
@{
    ViewBag.Title = "Home Page";
    Layout = null;
}

<script src="~/Scripts/jquery-1.10.2.min.js"></script>
<script src="~/Scripts/crossdomain.js"></script>

<div>
    <ul id="carList"></ul>
</div>
```
(Pro jednoduchost �pln� obch�z�me layoutovac� syst�m ASP.NET MVC a vypisujeme seznam na pr�zdnou str�nku.)

Kdybychom aplikaci nyn� spustili, zjist�me...
**+ App Web!**