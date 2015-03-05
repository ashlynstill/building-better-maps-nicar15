# Building better maps with Mapbox, Leaflet and Javascript


## Getting started

   
1. Create your Mapbox account at [mapbox.com](https://www.mapbox.com/)

	![sign up for mapbox](http://ashlynstill.github.io/nicar_maps/tipsheet_img/mapbox_signup.jpg)

2. Once you've filled out your info, grab your API key from your account homepage

	![sign up for mapbox](http://ashlynstill.github.io/nicar_maps/tipsheet_img/apikey.png)

3. We're ready to start building a basic map! Open the class folder in your text editor of choice, and specificially, open `index.html` for editing. This index file already has the Mapbox JS CDN and CSS CDN scripts included. Before we start writing any javascript to actually build our map, we have to add our Mapbox API key as a variable for Mapbox to use. Add a script with the following contents at the bottom of the body of your `index.html` file. Replace `YOUR MAPBOX API TOKEN HERE` with your api token from your mapbox account.

	````
	<body>



	<script>
		L.mapbox.accessToken = 'YOUR MAPBOX API TOKEN HERE';

	</script>
	</body>
	</html>
	````

	You'll be writing all of your code inside that script tag from now on, with the exception of a little bit of HTML and CSS, which I'll point out where that needs to go.

4. In the body of your `index.html` file, add a div and give it a specific id. We'll call ours `map`. Now our body looks like this (but with your mapbox api token filled in):

	````
	<body>

		<div id="map"></div>

	<script>
		L.mapbox.accessToken = 'MAPBOX API TOKEN';

	</script>
	</body>
	````

5. Now we're ready to render a simple map on our page. Let's make a map of the city of Atlanta, zoomed in to the I-285 perimeter. Add the following lines to your script, after your access token, to render the map. I've added comments into the code explaining what each line does.

	````
	<script>
		L.mapbox.accessToken = 'MAPBOX API TOKEN'; //access token
		
		var map = L.map('map', {center: [33.7550,-84.3900], zoom: 10, minZoom: 13 , maxZoom: 16});  //map variable to hold map object rendering inside #map div
		
		L.mapbox.tileLayer('mapbox.light').addTo(map); //this adds the base tile layer background (we've picked mapbox.light)
	</script>
	````

	Now, open your index.html file in any web browser and ta-da! You have a map of Atlanta. (*If you want to use a different base tile background layer, you can see more options [here](https://www.mapbox.com/developers/api/maps/)*). Leave your browser window open while we continue to work on this so we can see how our map is looking as we work on it.


## Adding a data layer

Having a map with nothing on it is boring and doesn't really tell us anything. Let's add some data! We're going to use crime data from the Atlanta Police Department for our mapping examples.

1. If you look at the top of your HTML file, you'll see a script tag linking to a `beats.js` file. This javascript file contains geojson shapes of each police beat in the city of Atlanta, held within a `beats` variable. Let's add these to our map.

	````
	<script>
		L.mapbox.accessToken = 'MAPBOX API TOKEN'; //access token
		
		var map = L.map('map', {center: [33.7550,-84.3900], zoom: 10, minZoom: 13 , maxZoom: 16}); 
		
		L.mapbox.tileLayer('mapbox.light').addTo(map);

		//load beats geojson here
		var beatsLayer = L.geoJson(beats).addTo(map);

	</script>
	````
	Refresh your browser window, and you should see blue beat shapes on your map!

2. Let's add some style to our beat shapes with a `beatsStyle` function. Adjust your `L.geoJson` call so it calls a `beatsStyle` function like below, then write out your `beatsStyle` function, passing `feature` in as a function argument. We'll also add in some temporary styles to play with.

	````
	//load beats geojson here
	var beatsLayer = L.geoJson(beats,{
		style: beatsStyle
	}).addTo(map);

	function beatsStyle(feature) {
		return {
	        weight: 1,
	        opacity: 1,
	        color: '#eee',
	        fillOpacity: 0.75,
	        fillColor: 'red'
	    };
	}
	````
	Refresh your browser. Your beats should now have red fills with much skinner white strokes.

3. But having all of our beats the same color doesn't tell us anything. If you go into your beats.js file in the data folder, you can see that each beat has the following properties:
	* beat
	* length
	* shape_area
	* zone
	* population
	* total_crime

	For our example, we're going to make a heat map where each beat is shaded according to number of crimes reported per capita (or, math speak, total_crime/population). To do this, we first need to add a function to calculate our color range.

	````
	function getColor(d) {
	    return d > 0.75 ? '#770702' :
	           d > 0.5  ? '#a6261e' :
	           d > 0.25 ? '#c95144' :
	        			  '#e37e73';
	}

	//load beats geojson here
	var beatsLayer = L.geoJson(beats,{
		style: beatsStyle
	}).addTo(map);
	````
	Then, we need to use our color range and the geojson properties to calculate what color the shape should be. We can grab each shape's properties via the `feature` argument we passed into the `beatsStyle` function. Here's how we calculate the color of the beat:

	````
	function beatsStyle(feature) {
	  	// calculate incidents per capita
	  	var percap = feature.properties.total_crime/feature.properties.population;

	    return {
	        weight: 1,
	        opacity: 1,
	        color: '#eee',
	        fillOpacity: 0.75,
	        fillColor: getColor(percap)  //changed from where it was just red earlier
	    };
	}
	````
	Save and refresh your browser again. Each beat should be shaded according to its crimes per capita value.

## Adding interactivity to your data layer

Now that you've got some data on your map, you need to add functionality so the user can interact with the map and explore the data.

1. First, we can call an `onEachFeature` function on our beatsLayer. This will render on each feature (in this case, beat shape) of the geojson layer. Then, we'll write a function called `onEachFeature`, which is where all our event-based functions will go.

	````
	// beats comes from the 'beats.js' script included above
	var beatsLayer = L.geoJson(beats,  {
	    style: beatsStyle,
	    onEachFeature: onEachFeature   //THIS IS NEW
	}).addTo(map);

	function beatsStyle(feature) {
	 	...
	};

	function onEachFeature(feature, layer) {
	    
	};
	````

2. Let's add three event functions, `highlightFeature`, `resetHighlight`, and `zoomToFeature`. These functions will make changes on mouseover, mouseout and click of each of the beat shapes. Just to make sure they work, let's add some `console.log()` statements in each function as well.
	````
	function onEachFeature(feature, layer) {
	    layer.on({
	        mouseover: highlightFeature,
	        mouseout: resetHighlight,
	        click: zoomToFeature
	    });
	};

	function highlightFeature(e) {
		console.log('entered '+e.target.feature.properties.beat);  //e.target acts same as 'layer'
	};

	function resetHighlight(e) {
		console.log('left '+e.target.feature.properties.beat);  //e.target acts same as 'layer'
	};

	function zoomToFeature(e) {
		console.log('clicked '+e.target.feature.properties.beat);  //e.target acts same as 'layer'
	}
	````

3. Log statements aren't very exciting, so let's make our functions actually do something visual. Let's update the feature style on mouseover, and reset on mouseout. Don't forget to take out the logs!

	````
	function highlightFeature(e) {
		var layer = e.target;

	    layer.setStyle({
	        weight: 3,
	        color: '#fff',
	        opacity:1,
	        fillOpacity: 0.9
	    });

	    if (!L.Browser.ie && !L.Browser.opera) {
	        layer.bringToFront();   // for your changes to move to the front. doesn't play nice in all browsers
	    }
	};

	function resetHighlight(e) {
		beatsLayer.resetStyle(e.target);
	};

	function zoomToFeature(e) {
	    map.fitBounds(e.target.getBounds());
	}
	````
	Refresh your browser and roll over your beats. Now they have some lovely hover and click functionality, and we've laid the foundations for future feature-specific events! Speaking of which...

## Adding a dynamic info box

Mapbox & Leaflet come with some nice legend and info box dom utilities out of the box, which is super helpful. We're going to use those to create an info box that shows us the crime per capita of your beat on hover.

1. First, let's add some CSS styles. Add the following after the styles for `#map` and `body` at the top of your index.html:

	````
	.info { 
    	position:absolute;
    	top:20px;
    	right:20px;
    	width:300px;
    	padding:10px;
    	background-color:#fff;
    	border:1px solid #ccc;
    	z-index:1000;
    }
    ````

2. Now we need to create the `.info` div with Javascript, and write two methods on the info box: one function that creates the div in the DOM when the L.control() object is added to the map object, and another function that inserts starting text after the div has been created. Then, at the end, you have to add it to the map.
	
	````
	// insert after base tile layer is added

	var info = L.control();

	info.onAdd = function(map) {
	    this._div = L.DomUtil.create('div', 'info'); // create a div with a class "info" once info control element is added to map
	    this.insertText();  //call insertText() after div is created
	    return this._div;
	};

	info.insertText = function(map) {
		this._div.innerHTML = '<h1>Crime in Atlanta</h1>'
			+ '<p>This map shows crime data from the Atlanta Police Department, from Jan 2009 - Feb 2015. The colors of the beats indicate the number of crimes per capita in that beat.</p><hr/>'
	    	+ '<h4>Hover over a beat</h4>';
		return this._div
	};

	info.addTo(map); //add to map
	````

	Refresh your browser and see what happens. The info box should appear, but nothing happens when you hover over the beats. So, what's next?

3. We need to write an updateText() method on info that is called on mouseover from the beatsLayer.

	````
	// after the other info methods but before you add info to map

	// method that we will use to update the control based on feature properties passed
	info.update = function(props) {
		this._div.children[3].innerHTML = '<h4>' +  (props ? 'Zone ' + props.zone + ', Beat '+ props.beat + '</h4><b>Crimes per capita:</b> ' + (props.total_crime/props.population).toFixed(2)
	        : 'Hover over a beat</h4>');
	};


	....


	function highlightFeature(e) {
	    var layer = e.target;

	    layer.setStyle({
	        weight: 3,
	        color: '#fff',
	        opacity:1,
	        fillOpacity: 0.9
	    });

	    if (!L.Browser.ie && !L.Browser.opera) {
	        layer.bringToFront();
	    }

	    info.update(layer.feature.properties); // ADD THIS to highlightFeature
	}

	function resetHighlight(e) {
	    beatsLayer.resetStyle(e.target);
	    info.update();  // ADD THIS to resetHighlight
	}
	````

Refresh your browser and roll over a few beats. You'll see the text in the info box change to show the data inside each feature. You can change the text to show whatever properties you wish in the `info.update` function.

## Adding a second data layer

If you look in your data folder, you'll see that there's also a `crimes.geojson` file. This geojson contains data for each violent crime reported from the week of Feb 13-21, 2015 (the most recent week reported by the APD). Luckily, the APD geocodes this data, so we're going to mark each of the violent crimes from that week on our map.

1. Instead of creating a geojson layer, we're going to create a Mapbox feature layer for this data.

	````
	// put this layer just after the beatsLayer so the crime markers render on top of the map and not under the beat shapes

	var crimeLayer = L.mapbox.featureLayer()
		.loadURL('data/crime.geojson')
		.addTo(map);
	````

2. But just have markers on a map doesn't tell the user much. Lets add popups with data attributes to our markers in our crimeLayer function.

	````
	var crimeLayer = L.mapbox.featureLayer()
		.loadURL('data/crime.geojson')
		.on('layeradd', function(e){ /// <<ADD THIS FUNCTION
			var marker = e.layer,
	        feature = marker.feature;

		    // Create custom popup content
		    var popupContent = '<h3>'+feature.properties.crime+'</h3><p>'+feature.properties.location+'</p><p>'+feature.properties.occur_date+'</p>';

		    // http://leafletjs.com/reference.html#popup
		    marker.bindPopup(popupContent,{
		        closeButton: true,
		        minWidth: 150
		 	})
		 })
		.addTo(map);
	````

Refresh the browser and click on a marker to see your popup!

3. Let's style the popup text a little bit with some CSS. If you use your browser's inspector (in Chrome, inspect element, or in Firefox, its best to use an extension like Firebug), you'll see that the popup lives in a div with the class `.leaflet-popup-content`. Let's style our `p` tags and `h3` tags so they fit better in the popup box.
	
	````
	<style type="text/css">

	...

	//after all other styles

	.leaflet-popup-content h3, .leaflet-popup-content p{
    	margin:0px;
    }
    .leaflet-popup-content p {
    	font-size:12px;
    }
    </style>
    ````

Refresh your page and click on a popup. Much better!

That's it for this course, but if you want to keep adding more to this map, the Mapbox API docs are extremely useful. There are a lot of great examples to play around with on the API doc site as well. Check the docs out here: [https://www.mapbox.com/mapbox.js/api/v2.1.5/](https://www.mapbox.com/mapbox.js/api/v2.1.5/).




