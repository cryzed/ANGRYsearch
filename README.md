# ANGRYsearch
Linux file search, instant results as you type

Attempt at making Linux version of [Everything Search Engine](https://www.voidtools.com/) because no one else bothered.  
Everyone seems to be damn content with searches that are slow, populating results as they go, or are cli based, or heavily integrated with a file manager, or limiting results to home, or are trying to be everything with full-text file's content search.

![demonstration gif](http://i.imgur.com/BsjGoYz.gif)

Done in python 3 using PyQt5 for GUI

### Lite mode vs Full mode

angrysearch can be set to two different modes in its config, default being `lite`
* **lite mode** shows only name and path
* **full mode** shows also size and date of the last modification, the drawback is that indexing takes roughly twice as long since every file and directory gets additional stats calls during indexing

in `~/.config/angrysearch/angrysearch.conf` you control the mode witht `angrysearch_lite` being set to true or false

![lite version png](http://i.imgur.com/TS1fgTr.png)


### What you should know:

* by default the search results are bound to the beginning of the words presented in the names  
  it would not find "Pi<b>rate</b>s" or "Whip<b>lash</b>", but it would "<b>Pir</b>ates" or "The-<b>Fif</b>th"  
  unchecking the checkbox in the top right corner fixes this, but searching gets slower
* the database is in `~/.cache/angrysearch/angry_database.db`  
  the config file is in `~/.config/angrysearch/angrysearch.conf`  
* if you have trouble starting the application, restart the pc, delete the database and the config file, new ones are recreated on the next run
* it can take ~2 min to index ~1 mil files(depending on hdd/ssd) and the database might be ~300MB
* it is not recommended to run as root, there's no reason for it and you might crawl where you would rather not, like Btrfs users going in to snapshots
* [xdg-open](https://wiki.archlinux.org/index.php/Default_applications#xdg-open) is used to open the files based on their mimetype, [default applications](http://i.imgur.com/u8jbi4e.png) can be set in `~/.local/share/applications/mimeapps.list` or `~/.config/mimeapps.list` 

### Installation:

* Arch Linux - [AUR package](https://aur.archlinux.org/packages/angrysearch/)
* openSUSE & Fedora [package](https://software.opensuse.org/package/angrysearch) (courtesy of [alanbortu](https://github.com/alanbortu))

![xubuntu installation demonstration](http://i.imgur.com/H9Uuxvp.png)

Manual installation is easy as there's no compilation with python, process consists of having dependencies, copying files somewhere and setting execution permissions  

**dependencies** - `python3-pyqt5`, `xdg-utils`  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;you need PyQt5 for python3, for example ubuntu based ditros: `sudo apt install python3-pyqt5`  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;most distros have xdg-utils out of the box  

Now download the latest [relase of angrysearch](https://github.com/DoTheEvo/ANGRYsearch/releases) and unpack it somewhere. Along the files there's one called `install.sh`, it will copy files where they belong and set correct permissions. I recommend having a look at it as you will be running it with sudo.

* open terminal in the directory with the release files
* set `install.sh` as executable and run it

        chmod +x install.sh
        sudo ./install.sh

* DONE, if you want to see more detailed instruction, [here](https://github.com/DoTheEvo/ANGRYsearch/tree/bf43e4e59da33ee3242d84074b0b4b3c9a6c9486#installation) is older version of this readme

**optional-dependancies**
  * [python3-gobject](https://wiki.gnome.org/Projects/PyGObject) - desktop notifications for automatic update, most DEs have it
  * [xdotool](https://www.semicomplete.com/projects/xdotool/xdotool.xhtml) - needed if using Thunar or PCmanFM and making use of the config option `fm_path_doubleclick_selects`

### Automatic update in the background

![notifications png](http://i.imgur.com/dudkCvZ.png)

Among the files there's `angrysearch_update_database.py`  
When this file is run there's no interface, it just crawls through drives, respecting ignored directories in the config, and updates the databse

Using [crontab](https://www.youtube.com/watch?v=UlVqobmcPuM) you can set this file to be executed periodicly at choosen intervals,
keeping angrysearch up to date with the changes on your system

* `crontab -l` - list cronjobs
* `crontab -e` - open text editor so you can enter new cronjob

this cronjob will execute the update every 6 hours

    0 */6 * * * /usr/share/angrysearch/angrysearch_update_database.py

Crontab does not try to catch up on a job if the PC has been off during scheduled time

`notifications` setting in the config turns on/off desktop notifications informing about background automatic update finishing  
`conditional_mounts_for_autoupdate` in the config can prevent autoupdate from running if set mount points are not present

### How it works & additional details:

![look in to the database](http://i.imgur.com/LuHZa3g.png)


* on update angrysearch crawls through your file system and creates a database in `~/.cache/angrysearch/angry_database.db`
* the database has two columns, one containing full path to every file or directory found, other column indicates if the path is to a file or a directory. If `full mode` is enabled then there are also columns for the last modification and for the size of files in bytes
* when typing in to the search input the full path column is searched for occurances of the searched terms and the one containing them are shown
* the database uses [FTS](https://sqlite.org/fts3.html) extension of sqlite for indexing to dramaticly improve search speed and get the instantaneous feel - results as you type
* drawback of this indexing is inability to do substring searches, but the checkbox in the top right corner can change this. If it's unchecked it will not use FTS tables and just do regular slow database search query
* **double-click** on items in search results:
  * `Name` - the first column, opens the file in application associated with its mimetype in xdg-open
  * `Path` - the second column, open the item's location in the file manager
* results can be sorted by clicking on column's headers, only the presented results will be sorted, meaning that by default max 500 items. To return to the default sort, sort by path column
* hotkeys
    * `F6` `ctrl+L` `alt+D` - focus search input
    * `Enter` - open selected item in associated application
    * `shift+Enter` - open items location
    * `Tab` - cycle through UI elements
    * `shift-Tab` - cycle backward through UI elements
    * `arrow up` `arrow down` - navigate through search results
    * `Esc` `ctrl+Q` - exit the application

### Configuration:

* **config file** location: `~/.config/angrysearch/angrysearch.conf`  
  You can delete the config file whenever you wish, on the next run/close a new one will be created with default values

![config file screenshot](http://i.imgur.com/KVPv3eV.png)

  * `angrysearch_lite` By default set to true. In lite mode theres only file name and path, no file size and no last modification date. Less informations but two times faster indexing of the drives
  * `conditional_mounts_for_autoupdate` By default empty. Purpose is to hold mount points that should be present when the database is being updated. If a mount is missing, automatic update through crontab will not run, but use system notification dialog to inform that paths set in this settings are not mounted. This prevents overwriting the databse when not all drives are present. Values are system mount points, space separated.
  * `darktheme` By default set to false. If set true dark theme is used for the applications interface, as defined in the qdarkstylesheet.qss, also resource_file.py contains icons for dark theme
  *   `directories_excluded` By default empty. Which directories to be ignored, directory names(no slashes) separated by space are valid value there. Can be set through program's interface, in the update window. Directory `proc` is hardcoded to be ignored
  *   `fast_search_but_no_substring` By default set to true. It holds the last set value of the checkbox affecting the speed of search and substrings, see FTS4 in the section above
  *   `file_manager` By default empty. Whatever application/script is put there, it receives the path when the path column is double-clicked. If left empty angrysearch will try to autodetect default file manager using xdg-utils. If one of the following file managers are set/detected: ['dolphin', 'nemo', 'nautilus', 'doublecmd'], the behaviour will change slightly, sending to those file managers full path to the file, highlighting the target file when opened in a filemanager.
  *   `fm_path_doubleclick_selects` By default set to false. Needs `xdotool` package, and preferably manualy set file manager in config. When set to true, Thunar, PCmanFM and SpaceFM file managers will be able to open containing directory with the file selected
  *   `icon_theme` By default set to adwaita. Which icon theme to use, can be set from program's interface in the update window. There are 6 icon types - folder, file, audio, image, video, text. Did not yet figure out how to get theme of the distro and reliably icon from file's mimetype, so packing icons with the angrysearch is the way
  *   `notifications` By default set to true. Automatic periodic updates that are run on background using crontab will use desktop notification system to inform when indexing is done or if the indexing was aborted because of missing mount points
  *   `number_of_results` By default set to 500. Limit set for searches in the database. Lower number means search results come faster
  *   `row_height` By default set to 0 which means default. Sets height of the rows in pixels
  *   `typing_delay` By default set to false. If enabled, it introduces 0.2 second delay between the action of typing and searching the database. This will prevent unnecessary database queries when user is typing fast as there is waiting to finish typing. This can improve performance on slower machines, but on modern ones it might negatively affect the feel of instant responsiveness
  *   `[Last_Run]` The applications properties from the last time at the moment when it was closed - window size, position, state

![dark theme screenshot](http://i.imgur.com/E3Bs5fx.png)
