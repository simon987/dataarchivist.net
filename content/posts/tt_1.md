---
title: "Web scraping with task_tracker"
date: 2019-06-14T14:31:42-04:00
draft: true
tags: ["scraping", "task_tracker"]
author: simon987
---

I built a tool to simplify long-running scraping tasks processes. **task_tracker** is a simple job queue
with a web frontend. This is a simple demo of a common use-case.

Let's start with a simple script I use to aggregate data from Spotify's API:

{{<highlight python >}}
import spotipy
from spotipy.oauth2 import SpotifyClientCredentials

spotify = spotipy.Spotify(...)


def search_artist(name, mbid):
	# Surround with quotes to get exact matches
    name = '"' + name + '"'

    with open(os.devnull, 'w') as null:
        # Silence spotipy's stdout...
        sys.stdout = null
        res = spotify.search(name, type="artist", limit=20)
	sys.stdout = sys.__stdout__

    with sqlite3.connect(dbfile) as conn:
        conn.execute("INSERT INTO artist (mbid, query, data) VALUES (?,?,?)", (mbid, name, json.dumps(res)))
        conn.commit()
{{</highlight>}}

I need to call `search_artist()` about 350000 times and I don't want to bother setting up multithreading, error handling and
keeping the script up to date on an arbitrary server so let's integrate it in the tracker.

My usual workflow is to create a project per script. I pushed the script to a [Gogs](https://gogs.io/) instance and created the project.
This also works with Github/Gitea.
{{< figure src="/tt/new_project.png" title="New task_tracker project">}}

After the Webhook is setup, **task\_tracker** will stay in sync with the repository its workers will be made aware of the new changes
instantly. This is not something we have to worry about since our **task_tracker_drone** takes care of deploying and updating the projects
in real time with no additional configuration.

{{< figure src="/tt/hook.png" title="Gogs webhook configuration">}}



{{<highlight python >}}
try:
	# This script is called like this:
	# python ./run.py "<recipe>" "<project_secret>"
    task_str = sys.argv[1]
    task = json.loads(task_str)
    secret_str = sys.argv[2]
    secret = json.loads(secret_str)

    CLIENT_ID = secret["CLIENT_ID"]
    CLIENT_SECRET = secret["CLIENT_SECRET"]
    client_credentials_manager = SpotifyClientCredentials(client_id=CLIENT_ID, client_secret=CLIENT_SECRET)
    spotify = spotipy.Spotify(client_credentials_manager=client_credentials_manager)

	# This job's recipe is an array of name & mbid pairs
    recipe = json.loads(task["recipe"])
    for job in recipe:
        search_artist(job["name"], job["mbid"])

except Exception as e:
    print(json.dumps({
		# Tell task_tracker that this task failed. It will be re-attempted later
        "result": 1,
		# Send full stack trace. It will be available in the Logs page.
        "logs": [
            {"message": str(e) + traceback.format_exc(), "level": 3}
        ]
    }))
    quit(2)

print(json.dumps({
    "result": 0,
}))
{{</highlight>}}




{{< figure src="/tt/secret.png" title="Project secret settings">}}
{{< figure src="/tt/perms.png" title="Private project require approval">}}
