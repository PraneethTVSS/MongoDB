# Basic Operators

## $match

- `$match` query is used to filter the input data.
- `$match` uses standard MongoDB read operation query syntax.
- Eg Query: db.coll.aggregate( [ { $match : {'type': {'$ne': 'Stars'} } } ] )
    - Here we are filtering the coll where the type is not Stars
- We cannot use $where operator in the $match query.
- $match should be at beginning of the pipeline.
- If this is at beginning, it makes use of indexes, we can see the speed of our queries.
- `$match` doesn't have any mechanism for projection.

## $project

- Shaping the documents.
- `$project` is used to retain and remove the fields.
- It is not only used to retains and remove, we can re-assign and and entirely to get new fields.

- Eg Query: db.coll.aggregate( [ { $project: {'_id': 0, 'name': 1} } ] )
    - Here the output will be the dictionaries with name
```bash
{'name': 'earth'}
{'name': 'jupiter'}
{'name': 'saturn'}
{'name': 'mars'}
```

- db.coll.aggregate( [ { $project: {'_id': 0, 'name': 1, 'gravity.value': 1 } } ] )
    - Here if we want to retain internal fields, we need to enclose with quotes.
```bash
{'name': 'earth', 'gravity.value': 9.8}
{'name': 'jupiter', 'gravity.value': 2.2}
{'name': 'saturn', 'gravity.value': 4.5}
{'name': 'mars', 'gravity.value': 6}
```
- Here we also can rename the fields i.e., project the field as a new field
db.coll.aggregate( [ { $project: {'_id': 0, 'name': 1, 'gravity': '$gravity.value' } } ] )
```bash
{'name': 'earth', 'gravity': 9.8}
{'name': 'jupiter', 'gravity': 2.2}
{'name': 'saturn', 'gravity': 4.5}
{'name': 'mars', 'gravity': 6}
```

- We want to perform some operations in the query.
- Lets say use `$divide`, `$multiply`.
```bash
db.coll.aggregate( [ {
      $project: {
        _id: 0,
        name: 1
        myWeight: { $multiply : [ { $divide: ["$gravity.value", 9.8] }, 50 ] } }
   }])
```

## $addFields

- `$addFields` are similar to `$project`. The name refers itself defines it returns the documents with new computed fields or modify the existing fields from incoming pipeline.
- It can add the fields at the higher level dict. We can re-assign the internal fields to the higher level dict.
- In general, if we want to get certain fields and remove some fields we do like below:
```bash
db.coll.aggregate( [ {$project: {_id: 0, name: 1, 'gravity': "$gravity.value", 'mass': "$mass.value"} } ] )
```
- If we use `$addFields` instead of `$project` fields like below:
```bash
db.coll.aggregate( [ {$addFields: {gravity: "$gravity.value", mass: "$mass.value"} } ] )
```

## $geoNear
- Used to perform geoqueries within the aggr pipeline. Baiscally it works with Geo-JSON data.
- $geoNear can be used on charted collections where $near can't.
- The collection which we are performing the geoNear operations, that should have one and only `2dsphere`.
- It should only have one index `geoIndex`.
- If we use `2dsphere`, the distance is returned in meters. If using legacy, the distance is returned in radians.
- `$geoNear` must be the first stage in an aggregation pipeline.
```
$geoNear: {
    near: {type: "Point", coordinates: [1, 2]},
    distanceField: <req field to insert in returned docs>,
    minDistance: <optional, in meters>,
    maxDistance: <optional, in meters>,
    query: <optional, allows querying source docs>,
    includeLocs: <optional, used to identify which location was used>,
    limit: <optional, the max docs to return>,
    num: <optional, same as limit>,
    spherical: <required, required to signal whether using a 2dsphere index>,
    distanceMultiplier: <optional, the factor to multiply all distances>
}
```
- near: is the point we'd like to search near, it returns the docs from closest to furthest from this location.
- distanceField: inserted into return docs giving us the distance from the location to location we specified in near. 
- spherical: true if 2dsphere index is present
- minDistance, maxDistance: closest, furthest point of data.
- query: it is like match.
- includeLocs: it would allow us to show what locations was used in the document if it has more than one location.
- limit, num: same
- distanceMultiplier: to convert distance results from radians to any other unit required

## Cursor like stages
### Methods are skip, limit, sort, counts.
- `count` returns the no of documents which matches our query.
- `skip` skips the first mentioned docs. Eg: `db.coll.find({}).skip(5)`
- `limit` returns the mentioned no of documents.
- `sort` returns the documents in sorted order. -1 ==> descending order.

### Stages: $limit, $sort, $count, $skip
- db.coll.aggregate( [ { $project: {_id: 0, name: 1}}, {$skip: 1} ] )
- sort is not limited, we can apply on multiple fields. Eg `db.coll.aggregate([{$project: {}, {$sort: {'field1': -1, 'field2': -1}}])`
- If the `sort` pipeline is near, it has great advantage of using indexes otherwise it uses high memory. For that cases, we need to allow another option called `allowDiskUse`. By default, the `$sort` query takes 100MB RAM.

## $sample
- Will select a set of random documents
- First method: `{ $sample: {size: <N, how many docs> }}`
    - When: N <= 5% of no of docs in source coll AND source coll >= 100 docs AND $sample is the first stage.
- Second method: `{ $sample: {size: <N, how many docs> }}`
