
# Mapzen Turn-by-Turn routing service API reference

Mapzen Turn-by-Turn, powered by the Valhalla engine, is an open-source routing service that lets you integrate routing and navigation into a web or mobile application. This page documents the inputs and outputs to the service.

The routing service is in active development. You can follow the [Mapzen blog](https://mapzen.com/blog) to get updates. To report software issues or suggest enhancements, open an issue in GitHub (use the [Thor repository](https://github.com/valhalla/thor) for comments about route paths or [Odin repository](https://github.com/valhalla/odin) for narration). You can also send a message to routing@mapzen.com.

The default logic for the OpenStreetMap tags, keys, and values used when routing are documented on an [OSM wiki page](http://wiki.openstreetmap.org/wiki/OSM_tags_for_routing/Valhalla).

## API keys and service limits

To use the Mapzen Turn-by-Turn routing service, you must first obtain an developer API key from Mapzen. Sign in at https://mapzen.com/developers to create and manage your API keys.

Mapzen Turn-by-Turn is a shared routing service. As such, there are limitations on requests, maximum distances, and numbers of locations to prevent individual users from degrading the overall system performance.

The following limitations are currently in place:

* Pedestrian routes have a limit of 50 locations and 250 kilometers.
* Bicycle routes have a limit of 50 locations and 500 kilometers.
* Automobile routes have a limit of 20 locations and 5,000 kilometers.
* Multimodal routes have a limit of 50 locations and 500 kilometers.

The distance limit is the total "as the crow-flies" distance along a path through successive locations.

Limits may be increased in the future, but you can contact routing@mapzen.com if you encounter rate limit status messages and need higher limits in the meantime.

## Inputs of a route

The route request takes the form of `valhalla.mapzen.com/route?json={}&api_key=`, where the JSON inputs inside the ``{}`` include location information, name and options for the costing model, and output options. Here is an example request:

`valhalla.mapzen.com/route?json={"locations":[{"lat":42.358528,"lon":-83.271400,"street":"Appleton"},{"lat":42.996613,"lon":-78.749855,"street":"Ranch Trail"}],"costing":"auto","costing_options":{"auto":{"country_crossing_penalty":2000.0}},"directions_options":{"units":"miles"}}&id=my_work_route&api_key=valhalla-xxxxxx`

This request provides automobile routing between the Detroit, Michigan area and Buffalo, New York, with an optional street name parameter to improve navigation at the start and end points. It attempts to avoid routing north through Canada by adding a penalty for crossing international borders. The resulting route is displayed in miles.

There is an option to name your route request.  You can do this by appending the following to your request `&id=`.  The `id` is returned with the response so a user could match to the corresponding request.

Note that you must append your own [API key](https://mapzen.com/developers) to the URL, following `&api_key=` at the end.

### Locations

You specify locations as an ordered list of two or more locations within a JSON array. Locations are visited in the order specified.  See the `API keys and service limits` section above for the locations and distance limits.

A location must include a latitude and longitude in decimal degrees. The coordinates can come from many input sources, such as a GPS location, a point or a click on a map, a geocoding service, and so on. Note that Mapzen Turn-by-Turn is a routing service only, so cannot search for names or addresses or perform geocoding or reverse geocoding. External search services, such as [Mapzen Search](https://mapzen.com/projects/search) or [Nominatim](http://wiki.openstreetmap.org/wiki/Nominatim), can be used to find places and geocode addresses, which must be converted to coordinates for input.  

To build a route, you need to specify two `break` locations. In addition, you can include `through` locations to influence the route path.

| Location parameters | Description |
| :--------- | :----------- |
| `lat` | Latitude of the location in degrees. |
| `lon` | Longitude of the location in degrees. |
| `type` | Type of location, either `break` or `through`. A `break` is a stop, so the first and last locations must be of type `break`. A `through` location is one that the route path travels through, and is useful to force a route to go through location. The path is not allowed to reverse direction at the through locations. If no type is provided, the type is assumed to be a `break`. |
| `heading` | (optional) Preferred direction of travel for the start from the location. This can be useful for mobile routing where a vehicle is traveling in a specific direction along a road, and the route should start in that direction. The `heading` is indicated in degrees from north in a clockwise direction, where north is 0°, east is 90°, south is 180°, and west is 270°. |
| `street` | (optional) Street name. The street name may be used to assist finding the correct routing location at the specified latitude, longitude. This is not currently implemented. |
| `way_id` | (optional) OpenStreetMap identification number for a polyline [way] (http://wiki.openstreetmap.org/wiki/Way). The way ID may be used to assist finding the correct routing location at the specified latitude, longitude. This is not currently implemented. |

Optionally, you can include the following location information without impacting the routing. This information is carried through the request and returned as a convenience.

* `name` = Location or business name. The name may be used in the route narration directions, such as "You have arrived at _&lt;business name&gt;_.")
* `city` = City name.
* `state` = State name.
* `postal_code` = Postal code.
* `country` = Country name.
* `phone` = Telephone number.
* `url` = URL for the place or location.
* `side_of_street` = (response only) The side of street of a `break` `location` that is determined based on the actual route when the `location` is offset from the street. The possible values are `left` and `right`.
* `date_time` = (response only) Expected date/time for the user to be at the location using the ISO 8601 format (YYYY-MM-DDThh:mm) in the local time zone of departure or arrival.  For example "2015-12-29T08:00".

Future development work includes adding location options and information related to time at each location. This will allow routes to specify a start time or an arrive by time at each location. There is also ongoing work to improve support for `through` locations.

### Costing models

Mapzen Turn-by-Turn uses dynamic, run-time costing to generate the route path. The route request must include the name of the costing model and can include optional parameters available for the chosen costing model.

| Costing model | Description |
| :----------------- | :----------- |
| `auto` | Standard costing for driving routes by car, motorcycle, truck, and so on that obeys automobile driving rules, such as access and turn restrictions. `Auto` provides a short time path (though not guaranteed to be shortest time) and uses intersection costing to minimize turns and maneuvers or road name changes. Routes also tend to favor highways and higher classification roads, such as motorways and trunks. |
| `auto_shorter` | Alternate costing for driving that provides a short path (though not guaranteed to be shortest distance) that obeys driving rules for access and turn restrictions. |
| `bicycle` | Standard costing for travel by bicycle, with a slight preference for using [cycleways](http://wiki.openstreetmap.org/wiki/Key:cycleway) or roads with bicycle lanes. Bicycle routes follow regular roads when needed, but avoid roads without bicycle access. |
| `bus` | Standard costing for bus routes. Bus costing inherits the auto costing behaviors, but checks for bus access on the roads. |
| `multimodal` | Currently supports pedestrian and transit. In the future, multimodal will support a combination of all of the above.  Here is an example multimodal request at the current date and time: `http://valhalla.mapzen.com/route?json={"locations":[{"lat":40.730930,"lon":-73.991379,"street":"Wanamaker Place"},{"lat":40.749706,"lon":-73.991562,"street":"Penn Plaza"}],"costing":"multimodal","directions_options":{"units":"miles"}}&api_key=valhalla-xxxxxxx`  Note that you must append your own [Mapzen Turn-by-Turn API key](https://mapzen.com/developers) to the URL, following `&api_key=` at the end. |
| `pedestrian` | Standard walking route that excludes roads without pedestrian access. In general, pedestrian routes are shortest distance with the following exceptions: walkways and footpaths are slightly favored, while steps or stairs and alleys are slightly avoided. |

#### Costing options

Costing methods can have several options that can be adjusted to develop the route path, as well as for estimating time along the path. Specify costing model options in your request using the format of `costing_options.type`, such as ` costing_options.auto`.

* Cost options are fixed costs in seconds that are added to both the path cost and the estimated time. Examples of costs are `gate_costs` and `toll_booth_costs`, where a fixed amount of time is added. Costs are not generally used to influence the route path; instead, use penalties to do this.
* Penalty options are fixed costs in seconds that are only added to the path cost. Penalties can influence the route path determination but do not add to the estimated time along the path. For example, add a `toll_booth_penalty` to create route paths that tend to avoid toll booths.
* Factor options are used to multiply the cost along an edge or road section in a way that influences the path to favor or avoid a particular attribute. Factor options do not impact estimated time along the path, though. Factors must be in the range 0.25 to 100000.0, where factors of 1.0 have no influence on cost. Use a factor less than 1.0 to attempt to favor paths containing preferred attributes, and a value greater than 1.0 to avoid paths with undesirable attributes. Avoidance factors are more effective than favor factors at influencing a path. A factor's impact also depends on the length of road containing the specified attribute, as longer roads have more impact on the costing than very short roads. For this reason, penalty options tend to be better at influencing paths.

##### Automobile and bus costing options

These options are available for `auto`, `auto_shorter`, and `bus` costing methods.

| Automobile options | Description |
| :-------------------------- | :----------- |
| `maneuver_penalty` | A penalty applied when transitioning between roads that do not have consistent naming–in other words, no road names in common. This penalty can be used to create simpler routes that tend to have fewer maneuvers or narrative guidance instructions. The default maneuver penalty is five seconds. |
| `gate_cost` | A cost applied when a [gate](http://wiki.openstreetmap.org/wiki/Tag:barrier%3Dgate) is encountered. This cost is added to the estimated time / elapsed time. The default gate cost is 30 seconds. |
| `toll_booth_cost` | A cost applied when a [toll booth](http://wiki.openstreetmap.org/wiki/Tag:barrier%3Dtoll_booth) is encountered. This cost is added to the estimated and elapsed times. The default cost is 15 seconds. |
| `toll_booth_penalty` | A penalty applied to the cost when a [toll booth](http://wiki.openstreetmap.org/wiki/Tag:barrier%3Dtoll_booth) is encountered. This penalty can be used to create paths that avoid toll roads. The default toll booth penalty is 0. |
| `ferry_cost` | A cost applied when entering a ferry. This cost is added to the estimated and elapsed times. The default cost is 300 seconds (5 minutes). |
| `use_ferry` | This value indicates the willingness to take ferries. This is range of values between 0 and 1. Values near 0 attempt to avoid ferries and values near 1 will favor ferries. The default value is 0.5. Note that sometimes ferries are required to complete a route so values of 0 are not guaranteed to avoid ferries entirely. |
| `country_crossing_cost` | A cost applied when encountering an international border. This cost is added to the estimated and elapsed times. The default cost is 600 seconds. |
| `country_crossing_penalty` | A penalty applied for a country crossing. This penalty can be used to create paths that avoid spanning country boundaries. The default penalty is 0. |

##### Bicycle costing options
The default bicycle costing is tuned toward road bicycles with a slight preference for using [cycleways](http://wiki.openstreetmap.org/wiki/Key:cycleway) or roads with bicycle lanes. Bicycle routes use regular roads where needed or where no direct bicycle lane options exist, but avoid roads without bicycle access. The costing model recognizes several factors unique to bicycle travel and offers several options for tuning bicycle routes. Several factors unique to travel by bicycle influence the resulting route.

*	The types of roads suitable for bicycling depend on the type of bicycle. Road bicycles (skinny or narrow tires) generally are suited to paved roads or perhaps very short sections of compacted gravel. They are not designed for riding on coarse gravel or most paths and tracks through wooded areas or farmland. Mountain bikes, on the other hand, are able to traverse a wider set of surfaces.
*	Average travel speed can be highly variable and can depend on bicycle type, fitness and experience of the cyclist, road surface, and hills. The costing model assumes a default speed on smooth, flat roads for each supported bicycle type. This speed can be overridden by an input option. The base speed is modulated by surface type (in conjunction with the bicycle type). Coming soon: logic to modify speed based on the hilliness of a road section.
*	Bicyclists vary in their tolerance for riding on roads. Most novice bicyclists, and even other bicyclists, prefer cycleways and dedicated cycling paths and would rather avoid all but the quietest neighborhood roads. Other cyclists may be experienced riding on roads and prefer to take roadways because they often provide the fastest way to get between two places. The bicycle costing model accounts for this with a `use_roads` factor to indicate a cyclist's tolerance for riding on roads.
*	Bicyclists vary in their fitness level and experience level, and many want to avoid hilly roads, and especially roads with very steep uphill or even downhill sections. Even if the fastest path is over a mountain, many cyclists prefer a flatter path that avoids the climb and descent up and over the mountain.

The following options described above for autos also apply to bicycle costing methods: `maneuver_penalty`, `gate_cost`, `gate_penalty`, `country_crossing_cost`, and `country_costing_penalty`.

These additional options are available for bicycle costing methods.

| Bicycle options | Description |
| :-------------------------- | :----------- |
| `bicycle_type` | The type of bicycle. <ul><li>`Road`: a road-style bicycle with narrow tires that is generally lightweight and designed for speed on paved surfaces. </li><li>`Hybrid` or `City`: a bicycle made mostly for city riding or casual riding on roads and paths with good surfaces.</li><li>`Cross`: a cyclo-cross bicycle, which is similar to a road bicycle but with wider tires suitable to rougher surfaces.</li><li>`Mountain`: a mountain bicycle suitable for most surfaces but generally heavier and slower on paved surfaces.</li><ul> |
| `cycling_speed` | Cycling speed is the average travel speed along smooth, flat roads. This is meant to be the speed a rider can comfortably maintain over the desired distance of the route. It can be modified (in the costing method) by surface type in conjunction with bicycle type and (coming soon) by hilliness of the road section. When no speed is specifically provided, the default speed is determined by the bicycle type and are as follows: Road = 25 KPH (15.5 MPH), Cross = 20 KPH (13 MPH), Hybrid/City = 18 KPH (11.5 MPH), and Mountain = 16 KPH (10 MPH). |
| `use_roads` | A cyclist's propensity to use roads alongside other vehicles. This is a range of values from 0 to 1, where 0 attempts to avoid roads and stay on cycleways and paths, and 1 indicates the rider is more comfortable riding on roads. Based on the `use_roads` factor, roads with certain classifications and higher speeds are penalized in an attempt to avoid them when finding the best path. |
| `use_hills` | A cyclist's desire to tackle hills in their routes. This is a range of values from 0 to 1, where 0 attempts to avoid hills and steep grades even if it means a longer (time and distance) path, while 1 indicates the rider does not fear hills and steeper grades. Based on the `use_hills` factor, penalties are applied to roads based on elevation change and grade. These penalties help the path avoid hilly roads in favor of flatter roads or less steep grades where available. Note that it is not always possible to find alternate paths to avoid hills (for example when route locations are in mountainous areas). |

##### Pedestrian costing options

These options are available for pedestrian costing methods.

| Pedestrian options | Description |
| :-------------------------- | :----------- |
| `walking_speed` | Walking speed in kilometers per hour. Defaults to 5.1 km/hr (3.1 miles/hour). |
| `walkway_factor` | A factor that modifies the cost when encountering roads or paths that do not allow vehicles and are set aside for pedestrian use. Pedestrian routes generally attempt to favor using these [walkways and sidewalks](http://wiki.openstreetmap.org/wiki/Sidewalks). The default walkway_factor is 0.9, indicating a slight preference. |
| `alley_factor` | A factor that modifies (multiplies) the cost when [alleys](http://wiki.openstreetmap.org/wiki/Tag:service%3Dalley) are encountered. Pedestrian routes generally want to avoid alleys or narrow service roads between buildings. The default alley_factor is 2.0. |
| `driveway_factor` | A factor that modifies (multiplies) the cost when encountering a [driveway](http://wiki.openstreetmap.org/wiki/Tag:service%3Ddriveway), which is often a private, service road. Pedestrian routes generally want to avoid driveways (private). The default driveway factor is 5.0. |
| `step_penalty` | A penalty in seconds added to each transition onto a path with [steps or stairs](http://wiki.openstreetmap.org/wiki/Tag:highway%3Dsteps). Higher values apply larger cost penalties to avoid paths that contain flights of steps. |

##### Transit costing options

These options are available for transit costing when the multimodal costing model is used.

| Transit options | Description |
| :-------------------------- | :----------- |
| `use_bus` | User's desire to use buses.  Range of values from 0 (try to avoid buses) to 1 (strong preference for riding buses).|
| `use_rail` | User's desire to use rail/subway/metro.  Range of values from 0 (try to avoid rail) to 1 (strong preference for riding rail).|
| `use_transfers` |User's desire to favor transfers.  Range of values from 0 (try to avoid transfers) to 1 (totally comfortable with transfers).|

For example, this is a route favoring buses, but also this person walks at a slower speed (4.1km/h)  `http://valhalla.mapzen.com/route?json={"locations":[{"lat":40.749706,"lon":-73.991562,"type":"break","street":"Penn Plaza"},{"lat":40.73093,"lon":-73.991379,"type":"break","street":"Wanamaker Place"}],"costing":"multimodal","costing_options":{"transit":{"use_bus":"1.0","use_rail":"0.0","use_transfers":"0.3"},"pedestrian":{"walking_speed":"4.1"}}}&api_key=valhalla-xxxxxxx`

Note that you must append your own [Mapzen Turn-by-Turn API key](https://mapzen.com/developers) to the URL, following `&api_key=` at the end.

#### Directions options

| Options | Description |
| :------------------ | :----------- |
| `units` | Distance units for output. Allowable unit types are miles (or mi) and kilometers (or km). If no unit type is specified, the units default to kilometers. |
| `language` | The language of the narration instructions based on the [IETF BCP 47](https://tools.ietf.org/html/bcp47) language tag string. If no language is specified or the specified language is unsupported, United States-based English (en-US) is used. Currently supported language tags: cs-CZ, de-DE, en-US, fr-FR, it-IT. |
| `narrative` |  Flag to allow users to disable narrative production. Locations, shape, length, and time are still returned. The narrative production is enabled by default. |

#### Other request options

| Options | Description |
| :------------------ | :----------- |
| `date_time` | This is the local date and time at the location.<ul><li>`type`<ul><li>0 - Current departure time.</li><li>1 - Specified departure time</li><li>2 - Specified arrival time. Not yet implemented for multimodal costing method.</li></ul></li><li>`value` - the date and time is specified in ISO 8601 format (YYYY-MM-DDThh:mm) in the local time zone of departure or arrival.  For example "2016-07-03T08:06"</li></ul><ul><b>NOTE: This option is not supported for our Matrix service.</b><ul> |
| `out_format` | Output format. If no `out_format` is specified, JSON is returned. Future work includes PBF (protocol buffer) support. |
| `id` | Name your route request. If `id` is specified, the naming will be sent thru to the response. |

This is an example of a transit route departing on 2016-03-29 at 08:00.
`http://valhalla.mapzen.com/route?json={"locations":[{"lat":40.749706,"lon":-73.991562,"type":"break","street":"Penn Plaza"},{"lat":40.73093,"lon":-73.991379,"type":"break","street":"Wanamaker Place"}],"costing":"multimodal","date_time":{"type":1,"value":"2016-03-29T08:00"}}&api_key=valhalla-xxxxxxx`

Note that you must append your own [Mapzen Turn-by-Turn API key](https://mapzen.com/developers) to the URL, following `&api_key=` at the end.

## Outputs of a route

If a route has been named in the request using the optional `&id=` input, then the name will be returned as a string `id` on the JSON object.

The route results are returned as a `trip`. This is a JSON object that contains details about the trip, including locations, a summary with basic information about the entire trip, and a list of `legs`.

Basic trip information includes:

| Trip Item | Description |
| :---- | :----------- |
| `status` | Status code. |
| `status_message ` | Status message. |
| `units` | The specified units of length are returned, either kilometers or miles. |
| `language` | The language of the narration instructions. If the user specified a language in the directions options and the specified language was supported - this returned value will be equal to the specified value. Otherwise, this value will be the default (en-US) language. |
| `locations` | Location information is returned in the same form as it is entered with additional fields to indicate the side of the street. |

The summary JSON object includes:

| Summary Item | Description |
| :---- | :----------- |
| `time` | Estimated elapsed time to complete the trip. |
| `length` | Distance traveled for the entire trip. Units are either miles or kilometers based on the input units specified. |

### Trip legs and maneuvers

A `trip` contains one or more `legs`. For *n* number of `break` locations, there are *n-1* legs. `Through` locations do not create separate legs.

Each leg of the trip includes a summary, which is comprised of the same information as a trip summary but applied to the single leg of the trip. It also includes a `shape`, which is an [encoded polyline](https://developers.google.com/maps/documentation/utilities/polylinealgorithm) of the route path, and a list of `maneuvers` as a JSON array. For more about decoding route shapes, see these [code examples](decoding.md).

Each maneuver includes:

| Maneuver Item | Description |
| :--------- | :---------- |
| `type` | Type of maneuver. See below for a list. |
| `instruction` | Written maneuver instruction. Describes the maneuver, such as "Turn right onto Main Street". |
| `verbal_transition_alert_instruction` | Text suitable for use as a verbal alert in a navigation application. The transition alert instruction will prepare the user for the forthcoming transition. For example: "Turn right onto North Prince Street". |
| `verbal_pre_transition_instruction` | Text suitable for use as a verbal message immediately prior to the maneuver transition. For example "Turn right onto North Prince Street, U.S. 2 22". |
| `verbal_post_transition_instruction` | Text suitable for use as a verbal message immediately after the maneuver transition. For example "Continue on U.S. 2 22 for 3.9 miles". |
| `street_names` | List of street names that are consistent along the entire maneuver. |
| `begin_street_names` | When present, these are the street names at the beginning of the maneuver (if they are different than the names that are consistent along the entire maneuver). |
| `time` | Estimated time along the maneuver in seconds. |
| `length` | Maneuver length in the units specified. |
| `begin_shape_index` | Index into the list of shape points for the start of the maneuver. |
| `end_shape_index` | Index into the list of shape points for the end of the maneuver. |
| `toll` | True if the maneuver has any toll, or portions of the maneuver are subject to a toll. |
| `rough` | True if the maneuver is unpaved or rough pavement, or has any portions that have rough pavement. |
| `gate` | True if a gate is encountered on this maneuver. |
| `ferry` | True if a ferry is encountered on this maneuver. |
| `sign` | Contains the interchange guide information at a road junction associated with this maneuver. See below for details. |
| `roundabout_exit_count` | The spoke to exit roundabout after entering. |
| `depart_instruction` | Written depart time instruction. Typically used with a transit maneuver, such as "Depart: 8:04 AM from 8 St - NYU". |
| `verbal_depart_instruction` | Text suitable for use as a verbal depart time instruction. Typically used with a transit maneuver, such as "Depart at 8:04 AM from 8 St - NYU". |
| `arrive_instruction` | Written arrive time instruction. Typically used with a transit maneuver, such as "Arrive: 8:10 AM at 34 St - Herald Sq". |
| `verbal_arrive_instruction` | Text suitable for use as a verbal arrive time instruction. Typically used with a transit maneuver, such as "Arrive at 8:10 AM at 34 St - Herald Sq". |
| `transit_info` | Contains the attributes that descibe a specific transit route. See below for details. |
| `verbal_multi_cue` | True if the `verbal_pre_transition_instruction` has been appended with the verbal instruction of the next maneuver. |
| `travel_mode` | Travel mode.<ul><li>"drive"</li><li>"pedestrian"</li><li>"bicycle"</li><li>"transit"</li></ul>|
| `travel_type` | Travel type for drive.<ul><li>"car"</li></ul>Travel type for pedestrian.<ul><li>"foot"</li></ul>Travel type for bicycle.<ul><li>"road"</li></ul>Travel type for transit.<ul><li>Tram or light rail = "tram"</li><li>Metro or subway = "metro"</li><li>Rail = "rail"</li><li>Bus = "bus"</li><li>Ferry = "ferry"</li><li>Cable car = "cable_car"</li><li>Gondola = "gondola"</li><li>Funicular = "funicular"</li></ul>|


For the maneuver `type`, the following are available:

```
kNone = 0;
kStart = 1;
kStartRight = 2;
kStartLeft = 3;
kDestination = 4;
kDestinationRight = 5;
kDestinationLeft = 6;
kBecomes = 7;
kContinue = 8;
kSlightRight = 9;
kRight = 10;
kSharpRight = 11;
kUturnRight = 12;
kUturnLeft = 13;
kSharpLeft = 14;
kLeft = 15;
kSlightLeft = 16;
kRampStraight = 17;
kRampRight = 18;
kRampLeft = 19;
kExitRight = 20;
kExitLeft = 21;
kStayStraight = 22;
kStayRight = 23;
kStayLeft = 24;
kMerge = 25;
kRoundaboutEnter = 26;
kRoundaboutExit = 27;
kFerryEnter = 28;
kFerryExit = 29;
kTransit = 30;
kTransitTransfer = 31;
kTransitRemainOn = 32;
kTransitConnectionStart = 33;
kTransitConnectionTransfer = 34;
kTransitConnectionDestination = 35;
kPostTransitConnectionDestination = 36;
```

The maneuver `sign` may contain four lists of interchange sign elements as follows:

* `exit_number_elements` = list of exit number elements. If an exit number element exists, it is typically just one value.
* `exit_branch_elements` = list of exit branch elements. The exit branch element text is the subsequent road name or route number after the sign.
* `exit_toward_elements` = list of exit toward elements. The exit toward element text is the location where the road ahead goes - the location is typically a control city, but may also be a future road name or route number.
* `exit_name_elements` = list of exit name elements. The exit name element is the interchange identifier - typically not used in the US.

Each maneuver sign element includes:

| Maneuver Sign Element Item | Description |
| :------------------ | :---------- |
| `text` | Interchange sign text. <ul><li>exit number example: 91B.</li><li>exit branch example: I 95 North.</li><li>exit toward example: New York.</li><li>exit name example: Gettysburg Pike.</li><ul> |
| `consecutive_count` | The frequency of this sign element within a set a consecutive signs. This item is optional. |

A maneuver `transit_info` includes:

| Maneuver Transit Route Item | Description |
| :--------- | :---------- |
| `onestop_id` | Global transit route identifier from Transitland. |
| `short_name` | Short name describing the transit route. For example "N". |
| `long_name` | Long name describing the transit route. For example "Broadway Express". |
| `headsign` | The sign on a public transport vehicle that identifies the route destination to passengers. For example "ASTORIA - DITMARS BLVD". |
| `color` | The numeric color value associated with a transit route. The value for yellow would be "16567306". |
| `text_color` | The numeric text color value associated with a transit route. The value for black would be "0". |
| `description` | The description of the the transit route. For example "Trains operate from Ditmars Boulevard, Queens, to Stillwell Avenue, Brooklyn, at all times. N trains in Manhattan operate along Broadway and across the Manhattan Bridge to and from Brooklyn. Trains in Brooklyn operate along 4th Avenue, then through Borough Park to Gravesend. Trains typically operate local in Queens, and either express or local in Manhattan and Brooklyn, depending on the time. Late night trains operate via Whitehall Street, Manhattan. Late night service is local". |
| `operator_onestop_id` | Global operator/agency identifier from Transitland. |
| `operator_name` | Operator/agency name. For example "BART", "King County Marine Divison", etc.  Short name is used over long name. |
| `operator_url` | Operator/agency URL. For example "http://web.mta.info/". |
| `transit_stops` | A list of the stops/stations associated with a specific transit route. See below for details. |

A `transit_stop` includes:

| Transit Stop Item | Description |
| :--------- | :---------- |
| `type` | The type of stop (simple stop=0; station=1) |
| `onestop_id` | Global transit stop identifier from Transitland. |
| `name` | Name of the stop/station. For example "14 St - Union Sq". |
| `arrival_date_time` | Arrival date/time using the ISO 8601 format (YYYY-MM-DDThh:mm). For example "2015-12-29T08:06". |
| `departure_date_time` | Departure date/time using the ISO 8601 format (YYYY-MM-DDThh:mm). For example "2015-12-29T08:06". |
| `is_parent_stop` | True if this stop is a marked as a parent stop. |
| `assumed_schedule` | True if the times are based on an assumed schedule because the actual schedule is not known. |
| `lat` | Latitude of the transit stop in degrees. |
| `lon` | Longitude of the transit stop in degrees. |

Continuing with the earlier routing example from the Detroit, Michigan area, a maneuver such as this one may be returned with that request: `{"begin_shape_index":0,"length":0.109,"end_shape_index":1,"instruction":"Go south on Appleton.","street_names":["Appleton"],"type":1,"time":0}`

In the future, look for additional maneuver information to enhance navigation applications, including landmark usage.

### Return codes and conditions

The following is a table of error conditions that may occur for a particular request. In general, the service follows the [HTTP specification](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes). That is to say that `5xx` returns are generally ephemeral server problems that should be resolved shortly or are the result of a bug. `4xx` returns are used to mark requests that cannot be carried out, generally due to bad input in the request or problems with the underlying data. A `2xx` return is expected when there is a successful route result or `trip`, as described above.

| Code | Body | Description |
| :--------- | :---------- | :---------- |
| 200 | *your_trip_json* | A happy bit of json describing your `trip` result |
| 400 | Failed to parse json request | You need a valid json request |
| 400 | Failed to parse location | You need a valid location object in your json request |
| 400 | Failed to parse correlated location | There was a problem with the location once correlated to the route network |
| 400 | Insufficiently specified required parameter 'locations' | You forgot the locations parameter |
| 400 | No edge/node costing provided | You forgot the costing parameter |
| 400 | Insufficient number of locations provided | You didn't provide enough locations |
| 400 | Exceeded max route locations of X | You are asking for too many locations |
| 400 | Locations are in unconnected regions. Go check/edit the map at osm.org | You are routing between regions of no connectivity |
| 400 | No costing method found for 'X' | You are asking for a non-existent costing mode |
| 400 | Path distance exceeds the max distance limit | You want to travel further than this mode allows |
| 400 | No suitable edges near location | There were no edges applicable to your mode of travel near the input location |
| 400 | No data found for location | There was no route data tile at the input location |
| 400 | No path could be found for input | There was no path found between the input locations |
| 404 | Try any of: '/route' '/locate' | You asked for an invalid path |
| 405 | Try a POST or GET request instead | We only support GET and POST requests |
| 500 | Failed to parse intermediate request format | Had a problem reading an intermediate request format |
| 500 | Failed to parse TripPath | Had a problem reading the computed path from Protobuf |
| 500 | Could not build directions for TripPath | Had a problem using the trip path to create TripDirections |
| 500 | Failed to parse TripDirections | Had a problem using the trip directions to serialize a json response |
| 501 | Not implemented | Not Implemented |
