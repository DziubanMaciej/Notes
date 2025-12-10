parentTopic:: [[Linux]]

# MIME type
Linux derives a MIME type for files and uses that to associate the file with a given application. MIME type isn't statically bound to a each file, but calculated each time.

## Querying MIME type
There are multiple tools to query the MIME. Unfortunately, they may sometimes return different results based on which directories they prioritize.
- `gio info FILENAME | grep content`
- `xdg-mime query filetype FILENAME`



/etc/mime.types


~/.config/mimeapps.list