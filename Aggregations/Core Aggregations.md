# Core Aggregations

## $group

- The field we should remember is _id. The expression or expressions we specify to _id becomes the criteria the group stage uses to categorize and bundle documents together.
- We can specify the different arguments in the group.
- Eg query:
```bash
{ $group: {_id: <matching/grouping criteria>,
     fieldName: <accumulator expression>,
     ....
  }
}
```
- Eg:
```bash
db.coll.aggregate([
  { 
    $group: {
      _id: "$field1",
      count: {$sum: 1}
  }
}])
```
- Everytime group categorizes a doc for us, the sum expression gets called.
- After grouping, sometimes we get `null` at the output. This is because, the field may be missing in the document itself.
- However we need to sanitize the incoming data and clean/filter the proper data.
- How to group all the docs instead of subset?
- Eg:
```bash
db.coll.aggregate([
  {
    $group: {
      _id: null or "",
      count: {$sum: 1}
    }
}])
```
- `$group` can be used multiple times in a pipeline.
- Another eg query for ref:
```bash
db.coll.aggregate([
  {
    $match: {'field1': {$gt: 0} },
  }, {
    $group: {
      _id: null,
      avg: {$avg: "$field1"},
    }
  }
])
```
- The above query is computing the average of `field1` for all the documents which are matching the given criteria.