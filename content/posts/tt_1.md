---
title: "Web scraping with task_tracker"
date: 2019-06-14T14:31:42-04:00
draft: true
author: simon987
---




{{<highlight python >}}
import spotipy
from spotipy.oauth2 import SpotifyClientCredentials

spotify = spotipy.Spotify(...)


def search_artist(name, mbid):
	# Surround with quotes to get exact matches
    name = '"' + name + '"'

    with open(os.devnull, 'w') as null:
        # Silence spotipy's stdout
        stdout = sys.stdout
        sys.stdout = null
        res = spotify.search(name, type="artist", limit=20)
        sys.stdout = stdout

    with sqlite3.connect(dbfile) as conn:
        conn.execute("INSERT INTO artist (mbid, query, data) VALUES (?,?,?)", (mbid, name, json.dumps(res)))
        conn.commit()
{{</highlight>}}


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




{{< figure src="/tt/new_project.png" title="New task_tracker project">}}
{{< figure src="/tt/secret.png" title="Project secret settings">}}
{{< figure src="/tt/hook.png" title="Gogs webhook configuration">}}
{{< figure src="/tt/perms.png" title="Private project require approval">}}
