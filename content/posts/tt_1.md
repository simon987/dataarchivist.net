---
title: "Web scraping with task_tracker"
date: 2019-06-14T14:31:42-04:00
draft: true
tags: ["scraping", "task_tracker"]
author: simon987
---

I built a tool to simplify long-running scraping tasks processes. **task_tracker** is a simple job queue
with a web frontend. This is a quick demo of a common use-case.

Let's start with a short script I use to aggregate data from Spotify's API:

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

    with psycopg2.connect(CONNSTR) as conn:
        conn.execute("INSERT INTO artist (mbid, query, data) VALUES (?,?,?)", (mbid, name, json.dumps(res)))
        conn.commit()
{{</highlight>}}

The `CONNSTR` variable is given 

I need to call `search_artist()` about 350'000 times and I don't want to bother setting up multithreading, error handling and
keeping the script up to date on an arbitrary server so let's integrate it in the tracker.

## Configurating the task_tracker project

My usual workflow is to create a project per script. I pushed the script to a [Gogs](https://gogs.io/) instance and created the project.
This also works with Github/Gitea.
{{< figure src="/tt/new_project.png" title="New task_tracker project">}}

After the Webhook is setup, **task\_tracker** will stay in sync with the repository its workers will be made aware of the new changes
instantly. This is not something we have to worry about since our **task_tracker_drone** takes care of deploying and updating the projects
in real time with no additional configuration.

{{< figure src="/tt/hook.png" title="Gogs webhook configuration">}}

The final configuration step is to set the *project secret*, which we will use to store authentication details.
Only workers which we have given explicit `ASSIGN` permission will be able to read this information.

{{< figure src="/tt/secret.png" title="Project secret settings">}}


## Writing the worker script

The way **task_tracker_drone** works is by passing the task object and project secret as command line arguments to the
 executable file called `run` in the root of the git repository. It also expects a json
 object telling it if the task was processed successfully, and if there are additionnal actions that needs to be executed:

**Expected result in stdout**:
{{<highlight json >}}
{
  "result": 1,
  "logs": [
    {"message": "This is an 'ERROR' (level 3) message that will be saved in the tracker", "level": 3}
  ],
  "tasks": [
    {
      "project": 1,
      "recipe": "This task will be submitted to the tracker",
      "..."
    },
  ]
}
{{</highlight>}}
*(See [LogLevel constants](https://github.com/simon987/task_tracker_drone/blob/master/src/tt_drone/api.py#L12))*


This is what the body of the final worker script looks like:

The program receives the task recipe and project secret as arguments, and it outputs the result object
to stdout.

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
    DB = secret["DB"]
    client_credentials_manager = SpotifyClientCredentials(client_id=CLIENT_ID, client_secret=CLIENT_SECRET)
    spotify = spotipy.Spotify(client_credentials_manager=client_credentials_manager)

	# This job's recipe is an array of name & mbid pairs
    recipe = json.loads(task["recipe"])
    for job in recipe:
        search_artist(job["name"], job["mbid"])

except Exception as e:
    print(json.dumps({
		# tell task_tracker that this task failed. it will be re-attempted later
        "result": 1,
		# send full stack trace. it will be available in the logs page.
        "logs": [
            {"message": str(e) + traceback.format_exc(), "level": 3}
        ]
    }))
    quit(2)

print(json.dumps({
    "result": 0,
}))
{{</highlight>}}


## Allocating worker machines

On the worker machines, you can execute the task runner and it will automatically start
working on the available projects. Private projects require explicit explicit approval to start executing tasks:

{{<highlight bash >}}
git clone https://github.com/simon987/task_tracker_drone
cd task_tracker_drone
python -m pip install -r requirements.txt

python ./src/drone.py "https://exemple-api-url.com/api" "worker alias"

# Request access for 1 r={"ok":true}
# Request access for 2 r={"ok":true}
# Request access for 3 r={"ok":true}
# Starting 10 working contexts
# No tasks, waiting...
{{</highlight>}}


{{< figure src="/tt/perms.png" title="Private projects require approval">}}

As soon as you give permission to the worker, it will automatically start executing tasks.
When a task fails, it will be put back in the task queue up to `task["max_retries"]` times.
The logs can be found on the web interface:

{{< figure src="/tt/logs.png" title="Logs page">}}
