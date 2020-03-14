# zoom-cloud-recording-downloader
Downloads Zoom cloud recordings to disk so you can archive them

### Usage

```
usage: zoomdl [-h] --token YOUR_JWT --start YYYY-MM-DD --end YYYY-MM-DD

Backs up Zoom cloud recordings

optional arguments:
  -h, --help          show this help message and exit
  --token YOUR_JWT    the JWT you generate from your Zoom app
  --start YYYY-MM-DD  first of month to start gathering recording meetings
  --end YYYY-MM-DD    first of month to end gathering recording meetings
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
