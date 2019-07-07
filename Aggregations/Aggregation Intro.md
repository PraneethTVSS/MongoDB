# Aggregation

- Aggregation is like a pipeline. In pipeline there are multiple stages.
- Lets assume there is a belt moving and it is having 3 stages. These stages transforms the input material to required output.
- Eg aggregation query: db.coll.aggregate( [ stage1, stage2, stageN ], {other options} )

- Other options are like "Allow disk use" etc.,
- There are many aggregation options. Ref: https://docs.mongodb.com/manual/meta/aggregation-quick-reference/
