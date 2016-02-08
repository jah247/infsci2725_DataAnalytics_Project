#Setting up the database
In this assignment, three raw data files were provided and one of them has a giant size of 200+ mb. After careful investigation of our task in this assignment, we chose to create three separate collections storing each raw data file to avoid data redundancy. Each data record was inserted into corresponding collection following the schema as in the original description readme document. Three python scripts were written to process each data file into a collection in a MongoDB database named MovieLens_database.

#Answering the queries
All queries (including 3 new queries we proposed) were successfully answered. Query 2, 4, 5 and our three queries all involve aggregation, which requires MongoDB version >= 2.1.0. As MongoDB does not support join operation, reference keys were first queried out and then used in find operation to answer query 2 and our proposed query a (see below). All answers to those queries can be found in **answer.txt**
##3 Different queries of our choice
    a. Which movie is rated most frequently?
    b. Who is the most active user?
    c. What is the most frequently used tag?

##All queries used to answer the questions (can be found in the **Answer.py** source file)
        client = pymongo.MongoClient()
        db = client['MovieLens_database']
        movies = db.movies
        ratings = db.ratings
        tags = db.tags

        movies.find({"Title": {'$regex': 'copycat', '$options': 'i'}}, {"_id": 0, "Genres": 1})

        pipeline2 = [
            {'$unwind': '$Genres'},
            {'$group': {'_id': '$Genres', 'count': {'$sum': 1}}},
            {'$sort': SON([('count', -1)])},
            {'$limit': 3}  # to ensure potential duplicates are also listed, use 3 instead of 1
        ]
        movies.aggregate(pipeline2)

        movieId = movies.find_one({"Title": {'$regex': '2001: A Space Odyssey', '$options': 'i'}})['MovieID']
        tags.find({"MovieID": movieId, "UserID": 146})

        pipeline4 = [
            {'$group': {'_id': '$MovieID', 'avg_rating': {'$avg': '$Rating'}}},
            {'$sort': SON([('avg_rating', -1)])},
            {'$limit': 5}
        ]
        ratings.aggregate(pipeline4)

        pipeline6a = [
            {'$group': {'_id': '$MovieID', 'count': {'$sum': 1}}},
            {'$sort': SON([('count', -1)])},
            {'$limit': 1}
        ]
        ratings.aggregate(pipeline6a)

        pipeline6b = [
            {'$group': {'_id': '$UserID', 'count': {'$sum': 1}}},
            {'$sort': SON([('count', -1)])},
            {'$limit': 1}
        ]
        ratings.aggregate(pipeline6b)

        pipeline6c = [
            {'$group': {'_id': '$Tag', 'count': {'$sum': 1}}},
            {'$sort': SON([('count', -1)])},
            {'$limit': 1}
        ]
        tags.aggregate(pipeline6c)

#Explanation on Python version
The **movies.dat** file contains lots of non-ascii characters and due to Python 2's notorious handling of unicode characters, generating the **answer.txt** using print statement and redirection always raises UnicodeEncodeError exception. To hack into this problem, we actually used Python 3' interpreter to run the script. To ensure minimal cross-compatibility, we imported `print_function` from the `__future__` package, which changes print statement to a function and allows `print` to work under Python 2 and Python 3.