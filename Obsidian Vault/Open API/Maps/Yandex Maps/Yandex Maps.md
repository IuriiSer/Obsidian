## Idea of usage
1 -> init map handler
2 -> join modules you need

## yMap handler
```
function initYMap() {
  return new Promise((res) => {
	// join all additional entitшуы tied to map
	// for example geo marks are already watched
	// and cache for geo marks
	if (!localStorage.viewedAdvs) localStorage.viewedAdvs = '';
	if (!localStorage.advsData) sessionStorage.advsData = '{}';
	
	ymaps.ready(init);
		
	function init() {
		const searchControl = new ymaps.control.SearchControl({
			options: {
				size: 'small',
				provider: 'yandex#search',
				noPlacemark: true,
				noPopup: true,
			},
		});
		
		const myMap = new ymaps.Map('map', {
			controls: ['routeButtonControl', searchControl],
				// join controlls you need
				// like routes, search, zoom and etc
			center: [55.83, 37.41],
				// center of map
			zoom: 10,
				// default zoom
		}, {
			searchControlProvider: 'yandex#search',
				// additional search settings -> search company or not
		});
		
		// objectManager <- tool to import a lot of geo data
		const objectManager = new ymaps.ObjectManager({
			clusterize: true,
			gridSize: 32,
			clusterDisableClickZoom: true, 
				// <- restrict to zoom cluster
			geoObjectOpenBalloonOnClick: false, 
				// <- restrict to open cluster Balloon
			clusterOpenBalloonOnClick: false, 
				// <- restrict to open cluster Balloon
		});

		// default pressets for placemark icons and cluster icon
		// -> color and styles
		objectManager
			.objects
			.options
			.set('preset', 'islands#greenDotIcon');
		objectManager
			.clusters
			.options
			.set('preset', 'islands#greenClusterIcons');

		
		myMap.geoObjects.add(objectManager);
		
		yMap.map = myMap;
		yMap.objectManager = objectManager;
		yMap.searchControl = searchControl;
		res();
	}
  });
}

// main object for map handler
// needed to join modules
const yMap = {
	isInited: initYMap(),
	map: undefined,
	objectManager: undefined,
	searchControl: undefined,
	visibleObjects: [],
};
```
## Modules
### addClickEventWithCallBackByCoords
```
async function addClickEventWithCallBackByCoords(yMap, callBack) {
	await yMap.isInited;
	yMap.map.events.add('click', (event) => {
		const coords = event.get('coords');
		if (callBack) callBack(coords);
	});
	return yMap;
}
```
### addDraggblePointWithCallBackByCoords
```
async function addDraggblePointWithCallBack(yMap, pointsCoords, callBack) {
	await yMap.isInited;
	
	yMap.map.geoObjects.removeAll();
		// ! this script REMOVE ALL MARKS
		
	const point = new ymaps.Placemark(pointsCoords, null, { draggable: true });
	
	point.events.add('dragend', (event) => {
		const coords = event.get('target').geometry.getCoordinates();
		if (callBack) callBack(coords);
	});
	
	yMap.map.geoObjects.add(point);
	
	return yMap;
}
```
### addNewObjsInMap
```
async function addNewObjsInMap(yMap, arrOfObjects) {
	await yMap.isInited;
	const objToAdd = {
		type: 'FeatureCollection',
		features: arrOfObjects,
	};
	yMap.objectManager.add(objToAdd);
	return yMap;
}
```
### dynamicBalloonWraper
```
function dynamicBalloonWraper(yMap) {
  yMap
    .objectManager
    .objects.events
	.add('click', async (event) => {
	
	// checking content in balloonContentBody
	function hasBalloonData(objectId) {
		const { balloonContentBody } = yMap
			.objectManager
			.objects
			.getById(objectId)
			.properties;
		return balloonContentBody;
	}
	
	// get Balloon id by event objectId
	// and get balloon obj from objectManager cache
	const id = event.get('objectId');
	const balloon = yMap.objectManager.objects.getById(id);
	
	// draw marker for object you alredy saw
	if (!localStorage.viewedAdvs.includes(id)) 
		localStorage.viewedAdvs += `${id},`;
	
	yMap
		.objectManager
		.objects
		.setObjectOptions(id, { preset: 'islands#yellowIcon' });
	
	// check did we have info in ballon
	// yes -> show balloon
	if (hasBalloonData(id)) 
		{ yMap.objectManager.objects.balloon.open(id); return; }
	
	// no -> load balloon content
	balloon.properties.balloonContentBody = 'Идет загрузка данных...';
	yMap.objectManager.objects.balloon.open(id);
	
	const { headerData, bodyData, footerData } = await renderMapSnippetMarkUp(id);
		// renderMapSnippetMarkUp <- func to fetch and convert balloon data
	balloon.properties.balloonContentHeader = headerData;
	balloon.properties.balloonContentBody = bodyData;
	balloon.properties.balloonContentFooter = footerData;
	// -> update balloon info in yndexMap cache
	yMap.objectManager.objects.balloon.setData(balloon);
  });
}
```
### getObjInMapArea
```
function getObjInMapArea(yMap, _callBack) {
	yMap.map.events.add('boundschange', () => {
		const callBack = _callBack;
		yMap.visibleObjects = ymaps
			.geoQuery(yMap.objectManager.objects)
			.searchIntersect(yMap.map);
		if (callBack) callBack(yMap);
	});
	return yMap;
}
```
## How to use
some component example
```
yMap.isInited.then(async () => {
	// field to show data
	const field = document.getElementById('address-input-coords');
	
	function writeCoordToField(coords) {
		field.value = coords.map((coord) => Number(coord.toFixed(3))).toString();
	}
	
	function createNewPointByCoords(coords) {
		addDraggblePointWithCallBack(yMap, coords, writeCoordToField);
		field.value = coords.map((coord) => Number(coord.toFixed(3))).toString();
	}
	
	addClickEventWithCallBack(yMap, createNewPointByCoords);
	addDraggblePointWithCallBack(yMap, field.value.split(','), writeCoordToField);
});
```