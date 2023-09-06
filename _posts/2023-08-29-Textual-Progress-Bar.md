---
layout: post
title: Textual Progress Bar
date: 2023-08-29
catagories: Textual, Python
---
I recently worked out how to get a progress bar in my Avocet bookmark app. Feel free to check it out if you want to see the full code (or even use it). I found that the first time the app loads can take a while, close to 30 seconds, and I didn't want anyone who uses it to think the it was stuck. After looking through the Textual documentation, I found that they already had a progress bar widget, so I didn't need to build it from scratch.

I use httpx for the calls to the Raindrop API, where I already story my bookmarks. I'm also using SQLAlchemy to store the bookmark data in a sqlite database, which is just a local file. I originally had some methods in the database manager class that looked like:

```python
async def add_collections(self):
    session = self.Session()
    raindrop_api = RaindropAPI()

    collection_data_list = await raindrop_api.get_collections()

    for collection_data in collection_data_list:
        collection = Collection(id=collection_data['_id'],
            title=collection_data['title'],
            description=collection_data["description"], 
            count=collection_data['count'], created=datetime.strptime(collection_data['created'], "%Y-%m-%dT%H:%M:%S.%fZ"), last_update=datetime.strptime(collection_data['lastUpdate'], "%Y-%m-%dT%H:%M:%S.%fZ"))

        session.add(collection)
        await self.add_raindrops(collection_data=collection, session=session, raindrop_api=raindrop_api)

    session.commit()
```

and

```python
async def add_raindrops(self, collection_data, session, raindrop_api):
    raindrop_data_list = await raindrop_api.get_raindrops_by_collection_id(collection_data.id)

    for raindrop_data in raindrop_data_list:
        raindrop = Raindrop(id=raindrop_data['_id'],
                            excerpt=raindrop_data['excerpt'],
                            link=raindrop_data['link'], created=datetime.strptime(raindrop_data['created'], "%Y-%m-%dT%H:%M:%S.%fZ"), last_update=datetime.strptime(raindrop_data['lastUpdate'], "%Y-%m-%dT%H:%M:%S.%fZ"),
                            collection_id=collection_data.id)
        session.add(raindrop)
        session.commit()
```

They were big and clunky, and called the Raindrop class to do the initial sqlite setup. For the progress bar, I obviously needed a way to measure progress. It dawned on me that the best way to do that would be to take the length of the collection list as the total, and for each collection the is entered into the database increment. A collection is like a bucket of raindrops (bookmarks).

I refactored these two database methods so that they added individual collections instead of all of them at once, and did the same for the raindrops. Now they look like this:

```python
def add_collection(self, collection_data):
    session = self.Session()
    collection = Collection(id=collection_data['_id'],
                            title=collection_data['title'], description=collection_data["description"], count=collection_data['count'], created=datetime.strptime(collection_data['created'], "%Y-%m-%dT%H:%M:%S.%fZ"), last_update=datetime.strptime(collection_data['lastUpdate'], "%Y-%m-%dT%H:%M:%S.%fZ"))

     session.add(collection)        
     session.commit()

def add_raindrops(self, raindrop_data_list, collection_id):
    session = self.Session()
    for raindrop_data in raindrop_data_list:
        raindrop = Raindrop(id=raindrop_data['_id'],
        excerpt=raindrop_data['excerpt'],
        note=raindrop_data['note'],
        title=raindrop_data['title'],
        link=raindrop_data['link'],
        created=datetime.strptime(raindrop_data['created'], "%Y-%m-%dT%H:%M:%S.%fZ"),
        last_update=datetime.strptime(raindrop_data['lastUpdate'], "%Y-%m-%dT%H:%M:%S.%fZ"),
        collection_id=collection_id)
        session.add(raindrop)
    session.commit()
```

My compose method, which is the entry point of the application, adds the progress bar and a label for it:

```python
def compose(self) -> ComposeResult:
    with Center():
        yield Label("Loading...")
        yield ProgressBar(total=100)
    with Center():
        yield OptionList(id="collection_option_list")
        yield OptionList(id="raindrop_option_list")
```

Then in the method that initializes the database I work my way through the collections and move the progress bar together:

```python
collection_data_list = await raindrop_api.get_collections()        self.query_one(ProgressBar).update(total=len(collection_data_list))
for collection_data in collection_data_list:
    self.database_manager.add_collection(collection_data)
    raindrop_data_list = await raindrop_api.get_raindrops_by_collection_id(collection_data["_id"])
    self.database_manager.add_raindrops(raindrop_data_list, collection_data["_id"])
    self.query_one(ProgressBar).advance(1)
```

Once it is finished and the method returns to the calling method, I can simply remove the progress bar and the label:

```python
self.query_one(ProgressBar).remove()
self.query_one(Label).remove()
```

A nice side benefit of having refactored the code to fit in the progress bar is that it seems to have sped up the initial startup by nearly half.
