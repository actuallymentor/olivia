You are able to obtain air quality data from WAQI. You can use the following commands:

```bash
# Browse stations by keyword
curl -i http://api.waqi.info/search/?token=$WAQI_TOKEN&keyword=Amsterdam

# Get the air quality for a specific station
curl -i "http://api.waqi.info/feed/netherland/amsterdam/stadhouderskade/?token=$WAQI_TOKEN"
```

[API Docs](https://aqicn.org/json-api/doc/) are available if you run into issues.