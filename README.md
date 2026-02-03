# zoom-cloud-recording-downloader
Downloads Zoom cloud recordings to disk so you can archive them

This repo contains hastily written python not meant for production use.

I wrote this to solve a problem not to launch a space shuttle. Instead of deleting it I'm putting it here.

### Installaion

```sh
$ pip install -r requirements.txt
```

### Usage

```
usage: zoomdl [-h] --token YOUR_JWT --start YYYY-MM-DD --end YYYY-MM-DD [--delete]

Backs up Zoom cloud recordings

optional arguments:
  -h, --help          show this help message and exit
  --token YOUR_JWT    the JWT you generate from your Zoom app
  --start YYYY-MM-DD  first of month to start gathering recording meetings
  --end YYYY-MM-DD    first of month to end gathering recording meetings
  --delete            delete recordings after downloading them
```


#### Example

```sh
$ ./zoomdl --start 2017-01-01 --end 2020-04-01 --token <your_jwt>
```

### Output

Current Working Directory
 - meetings.json
 - recordings/
    - `<recording uuid>.<file_type>`
