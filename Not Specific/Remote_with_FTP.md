# Remote with FTP

I like using FTP for file management on a site. I prefer it over using the file manager GUI of my hosting service. Unless I work with ZIP files, because unzipping a file on the server is faster, for me at least. But back to FTP.

My current workflow is building a site locally and upload it to the server (via FTP) when it’s done. Check if any functionality is broken due to the environment differences and fix those. So at the moment I don’t need to upload that often.

But recently I entered a project where I could not really do that without downloading thousands of files, which takes days. So I would have to style a page almost blindly, upload it and see what the result would be. Edit and upload again. That would be many uploads every minute that I would have to do manually. And I am not about doing stuff manually.

So I asked a colleague what their solution was, and a ftp plugin for your IDE was their answer. 
I went looking for a VS-Code extension and found [(this one)](https://marketplace.visualstudio.com/items?itemName=Natizyskunk.sftp).
It’s pretty good. Its documentation goes over a lot more than what I use and need at the moment. So I’m summarizing the options here in case I (inevitably) forget how it works. 😅

## Setup

Note that I am using the VS Code plugin, there will likely be differences if you use a different instance of this plugin for your own IDE.

For starters, you need to install the plugin (you can use the link above, or search for SFTP in the Extensions-tab in VS Code), and have a project folder ready.

```
- root
```

Go to your command pallet (`ctrl`+`shift`+`p` on Windows) and type in `SFTP: Config`.
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
You can add the `password` field if you want, otherwise you’d be prompted to fill in your password on connection.

The `remotePath` denotes the folder you wish to target, currently it’s set to the root. You could change it to only download a selected folder. Might be a smart idea if you have a site build with a file-happy CMS like Joomla or Wordpress.
Reopen your command pallet and now you could try connecting using the download command down below.
If nothing gets downloaded and you are sure your login data is correct, try changing the `"protocol": "sftp"` in the config file to `"protocol:" "ftp"`.

## Download

The following command fetches all the files in the targeted folder. `SFTP: Download Project`. They will be downloaded to the root folder. Use this to initiate your folder.
If no files present, nothing will be downloaded, so perhaps make connection using your ftp-application to add a test-file if none are present.
You could also use `SFTP: Sync Remote->Local` to do the same.

## Upload

The command `SFTP: Sync Local->Remote` will push any files in your root to the designated folder. It is smart enough to only sync the files that changed since the last upload.

## Boundaries

You might have noticed that the _.vscode_ folder also got uploaded, with our file with credentials. We don’t want that, so let’s ignore this folder when uploading. (Don’t forget to delete that file from your FTP.)
<small>root/.vscode/sftpconfig.json</small>

```json
"ignore": [".vscode"],
```

You could add any other folders you want to ignore, but in some cases that gets messy. For example, I like using node SASS and this means a lot of folders I don’t want to upload: the node-modules and any folders with SASS-files.
In that case we can constrict any fetch and push action to a single folder.
The rendered files get saved in a _public_ folder, or _live_ folder. Or whatever name you or your framework set it up to be.

By using the `context` option, we can border of a folder.
<small>root/.vscode/sftpconfig.json</small>
```json  
"context": "live" 
```

So only the contents of that folder get synced.

```
- root
	- live
	- .vscode
		- sftp.json
```

One thing I noticed, is that renaming files or folders does not play too well with this plugin. Instead of renaming, it creates a new file. So you would have to delete the old files. In that case I would simply delete the target folder with my ftp app, then sync my local environment to the server. This would ensure that no old remnants remain. It's a bit meh 🙄, but whatever. Overall it's still a better experience.

## In conclusion

<small>Copy the next snippet and replace the \<login credentials\> for a quick setup.</small>

```json
{
	"name": "My Project Server",
	"host": "<localhost>",
	"protocol": "<ftp>",
	"port": <21>,
	"username": "<username>",
	"password": "<password>",
	"remotePath": "/",
	"context": "live",
	"uploadOnSave": false,
	"useTempFile": false,
	"openSsh": false
}
```
