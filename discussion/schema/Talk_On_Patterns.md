# Talk on different MongoDB schema design patterns
There are multiple schema design patterns that could be applied for our project, but choosing the pattern which is efficient, cost-effective and memory efficient is hard to decide. Below are some of the schema design patters and their use cases. Our project could be treated as a project of IoT (Interet of Things) domain.


1. Approximation Pattern
2. Arrribute Pattern
3. Bucket Pattern
4. Computed Pattern
5. Document Versioning Pattern
6. Extended Reference Pattern
7. Outlier Pattern
8. Preallocated Pattern
9. Polymorphic Pattern
10. Schema Versioning Pattern
11. Subset Pattern
12. Tree and Graph Pattern


## Approximation Pattern
As the name suggests this pattern is handy when we want something "good enough" or the accuracy of the count is not mission critical. An example would be keeping track of people where the number is prety fluid as people move in and out of the city, some children is born and some people might die. So, here instead of writing to the database for every addition and deletion of an user we can count the number of users till it adds upto 100 and then update the number of users in the database, this will save the number of write operations that are performed on the database.

Why should we be concerned with this? Well, when working with large amounts of data or large numbers of users, the impact on performance of write operations can get to be large too. The more you scale up, the greater that impact is too and at scale, that's often your most important consideration. By reducing writes and reducing resources for data that doesn't need to be "perfect," it can lead to huge improvements in performance.

The Approximation Pattern is an excellent solution for applications that work with data that is difficult and/or expensive to compute and the accuracy of those numbers isn't mission critical. We can make fewer writes to the database increasing performance and still maintain statistically valid numbers. The cost of using this pattern, however, is that exact numbers aren't being represented and that the implementation must be done in the application itself.

## Attribute Pattern
The Attribute pattern is well suited when:

1. We have big documents with many similar fields but there is a subset of fields that share common characteristics and we want to sort or query on that subset of fields.
2. The fields we need to sort on are only found in a small subset of documents.
3. Both of the above conditions are met within the documents.

For performance reasons, to optimize our search we’d likely need many indexes to account for all of the subsets. Creating all of these indexes could reduce performance. The Attribute Pattern provides a good solution for these cases.

Consider the scenario where we are trying to fetch the release dates for different movies from the movies collection. The documents will likely have similar fields involved across all of the documents: title, director, producer, cast, etc. Let’s say we want to search on the release date. A challenge that we face when doing so, is which release date? Movies are often released on different dates in different countries.

```json
{
    title: "Star Wars",
    director: "George Lucas",
    ...
    release_US: ISODate("1977-05-20T01:00:00+01:00"),
    release_France: ISODate("1977-10-19T01:00:00+01:00"),
    release_Italy: ISODate("1977-10-20T01:00:00+01:00"),
    release_UK: ISODate("1977-12-27T01:00:00+01:00"),
    ...
}
```

A search for a release date will require looking across many fields at once. In order to quickly do searches for release dates, we’d need several indexes on our movies collection:

```json
{release_US: 1}
{release_France: 1}
{release_Italy: 1}
```

By using the Attribute Pattern, we can move this subset of information into an array and reduce the indexing needs. We turn this information into an array of key-value pairs:

```json
{
    title: "Star Wars",
    director: "George Lucas",
    …
    releases: [
        {
        location: "USA",
        date: ISODate("1977-05-20T01:00:00+01:00")
        },
        {
        location: "France",
        date: ISODate("1977-10-19T01:00:00+01:00")
        },
        {
        location: "Italy",
        date: ISODate("1977-10-20T01:00:00+01:00")
        },
        {
        location: "UK",
        date: ISODate("1977-12-27T01:00:00+01:00")
        },
        … 
    ],
    … 
}
```

By using the Attribute Pattern we can add organization to our documents for common characteristics and account for rare/unpredictable fields. For example, a movie released in a new or small festival. Further, moving to a key/value convention allows for the use of non-deterministic naming and the easy addition of qualifiers. For example, if our data collection was on bottles of water, our attributes might look something like:

```json
"specs": [
    { k: "volume", v: "500", u: "ml" },
    { k: "volume", v: "12", u: "ounces" }
]
```

Here we break the information out into keys and values, “k” and “v”, and add in a third field, “u” which allows for the units of measure to be stored separately.

```json
{"specks.k": 1, "specs.v": 1, "specs.u": 1}
```