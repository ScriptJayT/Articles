<!-- TODO: check json properties -->

# Remote with FTP

I like using FTP for filemanagement on a site. I prefer it over using the filemanager GUI of my hosting service. Unless I work with ZIP files, because unzipping a file on the server means faster launch, in my case. But back to FTP.

My current workflow is building a site localy and upload it to the server (via FTP) when it's done. Check if any functionality is broken due to the environment differences and fix those. So at the moment I don't need to upload that often.

But recently I entered a project where I could not realy do that. I would have to style a page almost blindly, upload it and see what the result would be. Edit and upload again. That would be many many uploads every minute and changes that I would have to do manually. And I am not about doing stuff manually.

So I asked a collegue what they used for this and they send me back the link to an SFTP plugin [(this one)](https://marketplace.visualstudio.com/items?itemName=Natizyskunk.sftp).

It's pretty good, though it's documentation is a bit scattered.
So I'm summarizing the options here, for me to find again when I inevitably forget in the future.

## Setup

I am using the VS Code plugin, so there will likely be differences if you use a different instance of this plugin for your own IDE.

For starters you need to install the plugin and have a project folder ready.

```
- root
```

Go to your command pallet (`ctrl`+`shift`+`p` on windows) and type in '`SFTP: Config`'.
This will create and open a _sftp.json_ file in a new folder in our root.

```
- root
	- .vscode
		- sftp.json
```

The file would look a bit like this:
<small>root/.vscode/sftp.json</small>

```json
{
	"name": "My Server",
	"host": "localhost",
	"protocol": "sftp",
	"port": 22,
	"username": "username",
	"remotePath": "/",
	"uploadOnSave": false,
	"useTempFile": false,
	"openSsh": false
}
```

You change the `host`, `username` and `port` to your own login data, provided by your host.
You can add the `password` field if you want, otherwise you'd be prompted to fill in your password on connection.

The `remotePath` denotes the folder you wish to target, currently it's set to the root. You could change it so you don't get all files downloaded. Might be a smart idea if you have a site build with a file-happy CMS like Joomla or Wordpress.
Reopen your command pallet and now you could try connecting using the download command down below.
If nothing gets downloaded and you are sure your login data is correct, try changing the `"protocol": "sftp"` in the config file to `"protocol:" "ftp"`.

## Download

The following command fetches all the files in the targeted folder. `SFTP: Download Project`. They will be downloaded to the root folder.
If no files present, nothing will be downloaded, so perhaps make connection using your ftp-application to add a test-file if none are present.

## Upload

The command `SFTP: Sync Local->Remote` will push any files in your root to the designated folder.

## Boundaries

You might have noticed that the _.vscode_ folder also got uploaded, with our file with credentials. We don't want that, so let's ignore this folder when uploading. (Don't forget to delete that file from your FTP.)
<small>/root/.vscode/sftpconfig.json</small>

```json
"ignore": [".vscode"],
```

You could add any other folders you want to ignore, but in some cases that gets messy. For example I like using node SASS and this means a lot of folders I don't want to upload, the node-modules and any folders with SASS-files as well.
In that case we constrict any fetch and push action to a single folder.
When using a compiler like Vite, most often the rendered files get saved in a _public_ folder, or _live_ folder. Or whatever name you set it up to be.

By using the `context` option, we can border of a folder.
<small>/root/.vscode/sftpconfig.json</small>

```json
"context": "live"
```

So only the contents of that folder get synched

```
- root
	- live
	- .vscode
		- sftp.json
```

## In conclusion

```json
{
	"name": "My Project Server",
	"host": "localhost",
	"protocol": "ftp",
	"port": 21,
	"username": "username",
	"password": "password",
	"remotePath": "/",
	"context": "live",
	"uploadOnSave": false,
	"useTempFile": false,
	"openSsh": false
}
```
