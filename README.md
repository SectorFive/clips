# Code Clips
I've been asked for some examples of code I commonly do.  I mostly handle the basics - form validation, content sliders, DOM manipulation, and maintaining state with the PHP session.  I'm not ready to host a workshop at An Event Apart just yet but in each project I build I try to branch out and learn something new.

---


#####Some examples of form validation I've written.  The first two Function Declarations display and remove warnings, and prevent duplicates.  Then the two Function Expressions perform the validation.
```javascript
// Add validation warnings
function markInvalid(fieldId, message) {
    $('#'+fieldId).siblings('.error').remove();  // Clears previous errors
    $('#'+fieldId).addClass('has-error');
    var errorText = '<label class="error">' + message + '</label>';
    $('#'+fieldId).after(errorText);
}
// Remove validation warnings
function removeInvalid(fieldId) {
    $('#'+fieldId).removeClass('has-error');
    $('#'+fieldId).siblings('.error').remove();
}

// Unique username validation from an ajax call
var validateUniqueName = function(elementId, message) {
	loginData = company_name.accountLogin_getItems(company_name.selectedAccount, null);  // ajax
	var totalNames = [];
	var possibleName = $('#userNameNew').val();
	$.each(loginData.data, function(i,v){
		totalNames.push(v.userName);
	});
	if (totalNames.indexOf(possibleName) == -1) {
		removeInvalid(elementId);
		return true;
	} else {
		markInvalid(elementId, message);
		return false;
	}
};

// Phone number validation using RegEx
var validatePhoneField = function(elementId) {
    var field = $('#'+elementId).val();
    var reg = /^\d+(?:-\d+)*$/;
    var validation = (reg.test(field)) ? true : false;
    return validation;
};
```


---

#####Here is an example of how you would use the above type of validation.  You call the validation function and pass in the ID of the element to be validated and the message if it fails:
```javascript
if (validateFieldEntered("userNameNew", "Please enter a username.") && validateUniqueName("userNameNew", "This username has already been registered.") && validateFieldEntered("pswdNew", "Please enter a password.") && validateSamePassword("pswdNew", "pswdNewAgain", "Passwords do not match.")) {
   ...
}
```

---


#####I decided to write a simple jQuery image-fader.  There are more efficient ones out there but it was a good little learning opportunity.

```javascript
// Content fader
$( document ).ready(function() {
    var slideNum = 0;
    var totalSlides = $(".slide").length;
    $(".slide").hide();
    $(".slide").eq(0).show();

    function fadeThem() {
        previousNum = slideNum;
        slideNum++;
        if (slideNum == totalSlides) {
            console.log("Resetting slideNum because it is " + slideNum + " and totalSlides is " + totalSlides);
            slideNum = 0;
        }
        $('.slide').css('z-index', "10");
        $('.slide').eq(previousNum).css('z-index', "15");
        $('.slide').eq(slideNum).hide().css('z-index', "20").fadeIn('slow');
    }
    // Timer
    slideInterval = setInterval(function() {
        fadeThem();
    }, 4000);
});
```


---


#####Here I used PHP to append ".." to a URL if it was being viewed on my development box because the folder structure in production was different:
```php
<?php
    // Add ".." to the src if this is on our webdev development server
    $url = 'http://' . $_SERVER['SERVER_NAME'] . $_SERVER['REQUEST_URI'];
    // Change "~cknoll" to your own webdev box name
    if (false !== strpos($url,'~cknoll')) {
        $urlPrepend = "..";
    } else {
        $urlPrepend = "";
    }
?>
<!-- Link to the API client -->
<script src="<?php echo $urlPrepend ?>/js/company_nameApi.js"></script>
```

---


#####A typical call to the API and then populating the page with the returned data for a "Class of Service" page using HTML data-types:
```javascript
function getList() {
    // Show loading text while data is retrieved
    $('#datalist').html('Loading Data...');
    // Retrieve data
    var new_api = company_name.cos_getList(null);  // an ajax call to the API
    // Display the data
    if (new_api.isError != null) {
        // Populate the page
        if (new_api.data != null) {
            // Clear anything previous
            $('#datalist').html('');
            $.each(new_api.data, function(i, v) {
                // If any are undefined, replace that value with a blank space
                $.each(v, function(index, val){
                    if(val == "undefined") new_api.data[i][index] = '';
                });
                // Create the data for the COSs
                var string = '';
                string += ('<tr>');
                string += ('<td>'+v.cosId+'</td>');
                string += ('<td>'+v.description+'</td>');
                string += ('<td>'+v.maxUp+'Mb <br/> '+v.maxDown+'Mb</td>');
                string += ('<td>'+v.guarantyUp+'Mb <br/>'+v.guarantyDown+'Mb</td>');
                string += ('<td>'+v.maxusers+'</td>');
                string += ('<td>'+v.cap+'</td>');
                string += ('<td class="table-action-hide"><a href="" data-target="#add-editbox" data-toggle="modal" class="edit-btn" style="opacity: 0;" data-cosid="'+v.cosId+'" data-description="'+v.description+'" data-maxup="'+v.maxUp+'" data-maxdown="'+v.maxDown+'" data-guaranteeUp="'+v.guaranteeUp+'" data-guaranteeDown="'+v.guaranteeDown+'" data-acl="'+v.acl+'" data-redirect="'+v.redirect+'" data-cap="'+v.cap+'" data-maxusers="'+v.maxusers+'"><i  class="fa fa-pencil"></i></a><a data-cosid="'+v.cosId+'" data-target="#del-box" data-toggle="modal" class="del-btn delete-row" href="" style="opacity: 0;"><i class="fa fa-trash-o"></i></a></td>');
                string += ('</tr>');
                // Put the data of each CoS into the #datalist table
                $('#datalist').append(string);
            });
            // Bind actions to buttons now that they have been created
            bindAllButtons();
        }
    }
}
```

---



#####A script that checks if this page was reached from a search that was performed:
```php
<?php if(isset($_GET['popsearchbox'])) {
    $popsearchbox = ($_GET['popsearchbox']);
    ?>
    // If they got here from the search bar at the top of the page
    $('#searchHeader').html('Search');
    // Populate with the term that was used in the search
    $('#acc').val("<?php echo $popsearchbox ?>");
    // Perform the search
    $('#searchBtn').click();
<?php } ?>
...
<?php if(isset($_GET['popsearchbox'])) {?>
	clickTheDropdown();
<?php } ?>
```

---



#####Using Google's GeoData to determine Longitude, Latitude and Timezone.  Uses a jQuery Local Storage plugin called jStorage:
```javascript
function updateGeoData(address, updateTZ) {
    var geocoder = new google.maps.Geocoder();
    var geoLoc = {};
    geocoder.geocode({
        'address': address
    }, function(results, status) {
        if (status == google.maps.GeocoderStatus.OK) {
            geoLoc.longitude = results[0].geometry.location.lng();
            geoLoc.latitude = results[0].geometry.location.lat();
            // Display results
            $('#ua_longitude').html(String(geoLoc.longitude).slice(0, 8));
            $('#ua_latitude').html(String(geoLoc.latitude).slice(0, 8));
            // jStore the above values for lat and lng
            $.jStorage.set("HHGProfile.longitude", String(geoLoc.longitude).slice(0, 8));
            $.jStorage.set("HHGProfile.latitude", String(geoLoc.latitude).slice(0, 8));
            $.jStorage.set("accountData.longitude", String(geoLoc.longitude).slice(0, 8));
            $.jStorage.set("accountData.latitude", String(geoLoc.latitude).slice(0, 8));
            if (updateTZ == true) {
                var timestamp = new Date().getTime();
                var tz = $.ajax({
                    url: 'https://maps.googleapis.com/maps/api/timezone/json?location=' + results[0].geometry.location.lat() + ',' + results[0].geometry.location.lng() + '&timestamp=1331161200&sensor=true_or_false'
                }).done(function(data) {
                    tzInfo = data;
                    $('#ua_timeZoneName').html(tzInfo.timeZoneName);
                    $('#ua_timeZoneId').html(tzInfo.timeZoneId);
                });
            }
        }
        return geoLoc;
    });
}
```
