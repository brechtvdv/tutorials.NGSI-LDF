# Install NGSI-LDF

```
git clone git@github.com:brechtvdv/ngsi-ldf.git
cd ngsi-ldf
npm install
npm start
```

# Fragmentations

NGSI-LDF currently supports two types of fragmentations:

**latest**:

With the latest fragmentation, only data that is modified during the last X minutes is returned.
Geospatially fragmentation (slippy tiles, geohash or H3) is provided with routes demonstrated below:

```
/:zoom/:tile_x/:tile_y/latest
/geohash/:hash/latest
/h3/:index/latest
```

**Event Stream**:

```
/:zoom/:tile_x/:tile_y
/geohash/:hash
/h3/:index
```


# Retrieve building with latest

You should be able to retrieve "De Krook" building of previous tutorial by looking up:
`http://localhost:3001/13/4180/2740/latest?type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw`

There is a `tree:AlternateViewRelation` relation provided to retrieve entities as an event stream.
