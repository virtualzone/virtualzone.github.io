---
title: "Determining a location’s federal state using Google Maps API"
date: 2012-08-10T11:30:03+00:00
tags:
    - google
    - api
author: "Heiner"
aliases:
    - /2012/08/determining-a-locations-federal-state-using-google-maps-api/
---

If you have to find out which federal state a city belongs to, you can use the Google Maps API v3. Here is a straightforward JavaScript code snippet:

```javascript
function log(s) {
    $('#sysout').append(document.createTextNode(s + 'n'));
}

function getResult(results) {
    for (var i=0; i -1) {
            return result['address_components'][j]['short_name'];
        }
    }
    return '';
}

function getCountry(result) {
    return extractFirst(result, 'country');
}

function getFederalState(result) {
    return extractFirst(result, 'administrative_area_level_1');
}

function searchLocation() {
    $('#sysout').empty();

    var location = $('#location').val();
    var geocoder;

    log('Looking up "' + location + '"');

    geocoder = new google.maps.Geocoder();
    geocoder.geocode({'address': location}, function(results, status) {
        if (status != google.maps.GeocoderStatus.OK) {
            log('error: ' + status);
            return;
        }
        if (results.length == 0) {
            log('no result');
            return;
        }

        log('Resolved to ' + results[0]['formatted_address']);

        var latlng = results[0]['geometry']['location'];
            geocoder.geocode({'latLng': latlng}, function(results, status) {
            if (status != google.maps.GeocoderStatus.OK) {
                log('error: ' + status);
                return;
            }
            var desiredResult = getResult(results);
            if (desiredResult) {
                log('Federal State: ' + getFederalState(desiredResult));
            }
        });
    });

    return false;
}

$(document).bind('ready', function() {
    new google.maps.places.Autocomplete(document.getElementById('location'), {});
    $('#form').submit(searchLocation);
});
```