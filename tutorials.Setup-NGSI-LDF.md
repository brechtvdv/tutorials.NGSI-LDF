# Install NGSI-LDF

```
git clone git@github.com:brechtvdv/ngsi-ldf.git
cd ngsi-ldf
npm install
npm start
```

# Retrieve building

You should be able to retrieve "De Krook" building of previous tutorial by looking up:
`http://localhost:3001/13/4180/2740/latest?type=https%3A%2F%2Fdata.vlaanderen.be%2Fns%2Fgebouw%23Gebouw`

There is a `tree:AlternateViewRelation` relation provided to retrieve entities as an event stream.
