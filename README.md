gitlab-discord-bridge
=====================

This is a super simple GitLab&gt;Discord bridge. It started out as a basic [standalone Python program](https://github.com/blha303/gitlab-discord-bridge), and then had Docker support added in a [fork](https://github.com/avluis/gitlab-discord-bridge). This is a fork of the latter, with modifications made to make it build and run on the `arm64v8` architecture.

Usage
-----

After setup (see below), the webhook URL is "http://&lt;ip&gt;:25431/&lt;discord channel id&gt;" to support posting in any channel. The bot user will need to be added to the server by an administrator before attempting to post.

Instructions
------------

Tested on Ubuntu 16.10 and OS X El Capitan (with Homebrew)

1. Install Python 3.5/3.6 (and Pip if not installed automatically): https://www.python.org/downloads/ or `brew install python3`
2. Clone or download the repository using the green button above.
3. Rename config.example.json to config.json and edit the file.
  * "secret" should be an arbitrary secret token that you'll be giving to Gitlab to make sure incoming requests to this bridge aren't from an attacker.
  * "token" should be a bot login token acquired after creating an application at https://discordapp.com/developers/applications/ and giving it a bot user.
4. Install the required libraries (Flask and requests): `sudo pip3 install -r requirements.txt`
5. Run the application with `python3 app.py`.
  * If you want to run it in the background with more worker threads, I use gunicorn: `sudo pip3 install gunicorn` `gunicorn -w 4 -b 0.0.0.0:25431 app:app`
  * You may need to forward port 25431 (TCP) if the server is behind a router. If the bridge and Gitlab are on the same machine or LAN disregard this.
6. In Gitlab, go into the desired repository's Webhooks settings and add "http://&lt;ip&gt;:25431/&lt;discord channel ID&gt;" as the webhook address. Set "Secret Token" to the same token as you set in config.json.
  * You can get the channel ID in the desktop client by opening User Settings &gt; Appearance &gt; Enable Developer Mode, then right clicking on the channel you want the bot to interact with and pressing "Copy ID". If using the browser client, the channel ID is the second string of numbers in the URL separated by forward slashes.
  * The bridge can currently handle these event types: push, tag push, issue, note, merge request and wiki page. The author doesn't use pipelines but would like to add support for them to the bot; if you use pipelines, enable that checkbox in Webhook settings and, after triggering a pipeline event, send the author the log from the server. All responses are treated as confidential and will be used to add pipeline support to future releases.
7. Test the webhook. You should see a message corresponding to the last event on the repository!

If you need assistance please [create an issue](https://github.com/blha303/gitlab-discord-bridge/issues).

Docker Instructions
-------------------

The Docker image follows a similar approach to a local install. Assuming you know how to use Docker, here is how to get started:

1. Create a directory to hold the application's `config.json` file.
2. Rename `config.example.json` to `config.json` and edit the file -- reference step 3 above.
3. Create the container with the following command:

```bash
docker create --name=gitlab-bridge \
-v <path to config file>:/usr/src/app/config.json \
-p 25431:25431 \
avluis/gitlab-discord-bridge
```

* Note: The application will not work without mounting the config file! If you encounter an error when running the container, verify your volume mount!