---
title: "Delete Route: Destroying a Resource"
slug: deleting-a-playlist
---

So we've come to the end of our RESTful and Resourceful routes. Only one to go: Delete.

| URL              | HTTP Verb | Action  |
|------------------|-----------|---------|
| /                | GET       | index   |
| /playlists/new     | GET       | new     |
| /playlists         | POST      | create  |
| /playlists/:id     | GET       | show    |
| /playlists/:id/edit     | GET       | edit    |
| /playlists/:id     | PUT/PATCH | update  |
| /playlists/:id     | DELETE    | Destroy |

# What the User Sees

As always, we start with what the users sees and does. So let's make a link to delete a playlist.

We can't set an `<a>` tag's method (it is always GET) so we are going to use a form to submit a DELETE request to our delete action path.

> [action]
>
> Add the delete button to `templates/playlists_show.html`:
>
```html
<!-- templates/playlists_show.html -->
{% extends 'base.html' %}
>
{% block content %}
<h1>{{ playlist.title }}</h1>
<h2>{{ playlist.description }}</h2>
{% for video in playlist.videos %}
    <iframe width='420' height='315' src='{{ video }}'></iframe>
{% endfor %}
>
<p><a href='/playlists/{{ playlist._id }}/edit'>Edit</a></p>
<!-- Delete button -->
<p><form method='POST' action='/playlists/{{ playlist._id }}/delete'>
    <button type='submit'>Delete</button>
</form></p>
{% endblock %}
```

Now we need a delete action route. After deleting the playlist, it should redirect to the home page (`playlists_index`).

> [action]
>
> Create the delete route in `app.py`:
>
```python
# app.py
...
@app.route('/playlists/<playlist_id>/delete', methods=['POST'])
def playlists_delete(playlist_id):
    """Delete one playlist."""
    playlists.delete_one({'_id': ObjectId(playlist_id)})
    return redirect(url_for('playlists_index'))
```

We did it! All **Resourceful Routes** for the `Playlist` resource are complete!

# Now Commit

> [action]
>
>
```bash
$ git add .
$ git commit -m "Users can destroy playlists"
$ git push
```
