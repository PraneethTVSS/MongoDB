# Basic Operators

## $match

- `$match` query is used to filter the input data.
- Eg Query: db.coll.aggregate( [ { $match : {'type': {'$ne': 'Stars'} } } ] )
    - Here we are filtering the coll where the type is not Stars
- We cannot use $where operator in the $match query.
- $match should be at beginning of the pipeline.

## $project

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

- `$addFields` are similar to `$project`. The name refers itself defines it returns the documents with new computed fields or modify the existing fields.
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
- The collection which we are performing the geoNear operations, that should have one and only `2dsphere`.
- If we use `2dsphere`, the distance is returned in meters. If using legacy, the distance is returned in radians.
- `$geoNear` must be the first stage in an aggregation pipeline.

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
