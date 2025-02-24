---
title: "Adding Tests"
slug: adding-tests
---

Automated testing is an important part of any professional coder's life. Writing automated tests slow you down to start, but then they speed you way up as your project gets more complex and large.

In Python, we will be using the **unittest** Unit Testing Framework to run our tests and **unittest.mock** to mock out some sample data. These come packaged with Python 3, so there are no special install instructions.

# What is Automated Testing?

Automated testing is code that tests if your code is working. The opposite is **Manual Testing**.

No matter what you have to test your code and you have to test that your new code didn't break older code. If you have automated tests set up, then you can just run the tests for old code to make sure that your new code didn't break any old code. This means that old tests become **Regression Tests**, and act as a sort of double check that new code didn't break old stuff.

So the tests you write today for your new feature will eventually become regression tests. This helps us to make sure that your feature doesn't break when someone adds to your code.

# Setting Up The Unittest Framework

We'll start by setting up a `TestCase` class and adding a method to test the index route of our Playlist resource.

> [action]
> Create a file in your project folder called `tests.py` to put the test code in.
>
> Next, we'll need to create a test class to hold the unit tests.
>
```python
# tests.py
>
from unittest import TestCase, main as unittest_main
from app import app
>
class PlaylistsTests(TestCase):
    """Flask tests."""
>
    def setUp(self):
        """Stuff to do before every test."""
>
        # Get the Flask test client
        self.client = app.test_client()
>
        # Show Flask errors that happen during tests
        app.config['TESTING'] = True
>        
if __name__ == '__main__':
    unittest_main()
```

Let's run through a quick overview of what this code does:

* From the `unittest` module, we are importing `TestCase`, which our class is a subclass of, as well as the `main` function, which tells Python how to run our tests.
* The `PlaylistsTests` class extends `TestCase` and will hold all of our unit tests of the Playlist resource.
* The `setUp` method will automatically run before every test. Here, we are setting up a test Flask client and setting its `TESTING` field to True so that we can see any Flask errors.
* Finally, we tell Python to run unittest's `main` function when we run this program.

To verify that our file is set up correctly, let's try running it!

> [action]
> Go to your terminal window inside the project directory, and run the following:
>
```bash
(env) $ python3 tests.py
```

What do you see? You should get some output that looks like:

```bash
----------------------------------------------------------------------
Ran 0 tests in 0.000s

OK
```

Next up: Adding some tests!

# Adding One Test: Index Route

Now we'll add a test for the index route of our Playlist resource.

To check whether we receive data correctly from the test Flask client, we will be using **Asserts**. Asserts test whether the output from our program (in this case, the Flask client) matches what we expect to see. Since we're using the `unittest` framework, there are many different assert methods that we can choose from, including `assertEqual` and `assertIn`. Check out the [unittest documentation](https://docs.python.org/3/library/unittest.html#unittest.TestCase) for more information.

In our case, we want to see that the index page loads with 1) a status of 200 and 2) the data - in this case, text - that we expect to see. We can use asserts to do that!

> [action]
> Add the following to `tests.py` after the `setUp` method, inside of the `PlaylistsTests` class:
>
```python
# tests.py
class PlaylistsTests(TestCase):
    ...
    def test_index(self):
        """Test the playlists homepage."""
        result = self.client.get('/')
        self.assertEqual(result.status, '200 OK')
        self.assertIn(b'Playlist', result.data)
```
>
> Now go to your terminal window and run your tests again.
>

You should now see something like:

```bash
.
----------------------------------------------------------------------
Ran 1 test in 0.011s

OK
```

# Red-Green-Refactor

Before moving on, we should double check to see if we can break the test and make it return false.

> [action]
>
> Set the test to expect the response to have a status of `404` and rerun your tests. Did it turn "Red"?

What was the error message?

Now put it back to 200 and see if it is green again.

If you don't do this step with tests, you might just be testing an **Identity** like that `4 == 4`. That's not a very good test!

# Next Test: New

Now that we have the Index route, let's use the same sort of testing logic for the New route.

> [action]
>
> Add the following test to `tests.py`:
>
```python
class PlaylistsTests(TestCase):
...
    def test_new(self):
        """Test the new playlist creation page."""
        result = self.client.get('/playlists/new')
        self.assertEqual(result.status, '200 OK')
        self.assertIn(b'New Playlist', result.data)
```

Now run your tests!

# Next Test: Show & Edit

To test the rest of the tests, we'll need to create some sample data. We don't want to use the actual data that's on our PyMongo server for our tests, because if the data were to change because someone deleted a playlist, it would break our tests! Instead, we'll be using `unittest.mock` to send mock data to our routes.

> [action]
> Modify your import statement to import the mock object library into your test file, and add a sample playlist at the top of the file, before the PlaylistsTests class.
>
```python
from unittest import TestCase, main as unittest_main, mock
from bson.objectid import ObjectId
>
sample_playlist_id = ObjectId('5d55cffc4a3d4031f42827a3')
sample_playlist = {
    'title': 'Cat Videos',
    'description': 'Cats acting weird',
    'videos': [
        'https://youtube.com/embed/hY7m5jjJ9mM',
        'https://www.youtube.com/embed/CQ85sUNBK7w'
    ]
}
sample_form_data = {
    'title': sample_playlist['title'],
    'description': sample_playlist['description'],
    'videos': '\n'.join(sample_playlist['videos'])
}
```

Now we can use the mock **decorator** to tell our test function that we'll be using a fake version of PyMongo's `find_one` operation. We can set its return value to our sample playlist, and voila! We'll see that sample data in the Flask result for the `playlists_show` route.

> [action]
> Add a show test to `tests.py`:
>
```python
class PlaylistsTest(TestCase):
...
    @mock.patch('pymongo.collection.Collection.find_one')
    def test_show_playlist(self, mock_find):
        """Test showing a single playlist."""
        mock_find.return_value = sample_playlist
>
        result = self.client.get(f'/playlists/{sample_playlist_id}')
        self.assertEqual(result.status, '200 OK')
        self.assertIn(b'Cat Videos', result.data)
```
>
> The edit route is the same. Add that now:
>
```python
    @mock.patch('pymongo.collection.Collection.find_one')
    def test_edit_playlist(self, mock_find):
        """Test editing a single playlist."""
        mock_find.return_value = sample_playlist
>
        result = self.client.get(f'/playlists/{sample_playlist_id}/edit')
        self.assertEqual(result.status, '200 OK')
        self.assertIn(b'Cat Videos', result.data)
```

# Next Test: Create

Now we'll do the create route test. We'll send in the `sample_playlist` as JSON to the route and check that the response is OK. Note that this time, we are checking for a `302 FOUND` response code instead of `200 OK`. This is because we are expecting the page to be redirected.

We can also verify that the data we send is inserted into PyMongo with `assert_called_with`.

> [action]
>
> Add a create test to 'tests.py':
>
```python
    @mock.patch('pymongo.collection.Collection.insert_one')
    def test_submit_playlist(self, mock_insert):
        """Test submitting a new playlist."""
        result = self.client.post('/playlists', data=sample_form_data)
>
        # After submitting, should redirect to that playlist's page
        self.assertEqual(result.status, '302 FOUND')
        mock_insert.assert_called_with(sample_playlist)
```

Run your tests again! Your line of green dots should be getting longer. Can you make this create route test fail and turn red and then turn it back to green?

# Next Test: Update

For update we have to create the sample playlist, then send in a PUT message with a change and see if it response with a 200.

> [action]
>
> Add an update test to 'tests.py':
>
```python
    @mock.patch('pymongo.collection.Collection.update_one')
    def test_update_playlist(self, mock_update):
        result = self.client.post(f'/playlists/{sample_playlist_id}', data=sample_form_data)
>
        self.assertEqual(result.status, '302 FOUND')
        mock_update.assert_called_with({'_id': sample_playlist_id}, {'$set': sample_playlist})
```

# Next Test: Delete

Finally we have to see if we can test deleting a playlist. We'll create the sample playlist, then send in a delete message to see if we can destroy it and get a 302 status back.

> [action]
>
> Add a delete test to `tests.py`:
>
```python
    @mock.patch('pymongo.collection.Collection.delete_one')
    def test_delete_playlist(self, mock_delete):
        form_data = {'_method': 'DELETE'}
        result = self.client.post(f'/playlists/{sample_playlist_id}/delete', data=form_data)
        self.assertEqual(result.status, '302 FOUND')
        mock_delete.assert_called_with({'_id': sample_playlist_id})
```

# Now Commit

Good work adding your Resourceful Routes test sweet. Your product currently has **100% Test Coverage** because each route has a corresponding test.

Now commit these changes and push them to github:

> [action]
>
>
```bash
$ git add .
$ git commit -m 'Added tests'
$ git push
```
