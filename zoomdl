#!/usr/bin/env python3

import json
import os
import argparse
import requests
import datetime

# Zoom only allows specifying a range of 1 month max in recordings queries
def make_months(start_date, end_date):
    # The api docs say you can only go back 6 months but that isn't true
    dt_format = '%Y-%m-%d'
    dt_start = datetime.datetime.strptime(start_date, dt_format)
    dt_end = datetime.datetime.strptime(end_date, dt_format)
    one_day = datetime.timedelta(1)

    start_dates = [dt_start.strftime(dt_format)]
    end_dates = []
    today = dt_start

    while today <= dt_end:
        tomorrow = today + one_day

        if tomorrow.month != today.month:
            start_dates.append(tomorrow.strftime(dt_format))
            end_dates.append(today.strftime(dt_format))

        today = tomorrow

    end_dates.append(dt_end.strftime(dt_format))

    months = zip(start_dates,end_dates)

    return months

def make_headers(token):
    return {
        "Authorization": f"Bearer {token}",
        "User-Agent": "Zoom-api-Jwt-Request",
        "content-type": "application/json"
    }

def get_meetings_in_month(headers, meetings, month_start, month_end, tok=None):
    params = {
        'from': month_start,
        'to': month_end,
    }

    if tok:
        params['next_page_token'] = tok

    result = requests.get(
        f'https://api.zoom.us/v2/accounts/me/recordings',
        headers=headers,
        params=params
    )

    # token could be bad or expired, throw
    result.raise_for_status()

    output = result.json()
    meetings.extend(output['meetings'])

    num_recs = len(output['meetings'])
    print(f'got {num_recs} recorded meetings')

    total_meetings = len(meetings)
    print(f'found {total_meetings} recorded meetings so far...')

    next_tok = output['next_page_token']
    if next_tok != '':
        get_meetings_in_month(headers, meetings, month_start, month_end, next_tok)

def get_meetings(headers, months):
    meetings = []

    # If we've already done the work, just load the file
    if os.path.exists('meetings.json'):
        with open('meetings.json') as meetings_file:
            for line in meetings_file:
                meetings.append(json.loads(line))

    else:
        for month in months:
            print(f'Getting meetings for {month}')
            get_meetings_in_month(
                headers,
                meetings,
                month_start=month[0],
                month_end=month[1]
            )

        # This file will serve as our manifest, keep it with the recordings
        with open('meetings.json', 'w') as meetings_file:
            for meeting in meetings:
                meetings_file.write(json.dumps(meeting) + '\n')

    return meetings

def get_recordings(recordings):
    # Each meeting can have multiple recordings (chat, audio, video) and
    # if meetings are long enough you can have multiple video segments
    for recording in recordings:
        recording_id = str(recording['id'])
        file_name = recording_id + "." + recording['file_type']
        file_path = os.path.join('recordings', file_name)

        # Allows us to resume work in case we get interrupted
        if os.path.exists(file_path):
            print(f'already have {file_path}')
            continue

        print(f'downloading {file_path}')

        result = requests.get(recording['download_url'], stream=True)

        with open(file_path, 'wb') as raw:
            for chunk in result.iter_content(1024):
                if not chunk:
                    break

                raw.write(chunk)

def delete_recordings(recordings, headers):
    # This could take meetings and delete them wholesale but it
    # seems like a good check to delete them from the manifest
    # individually so we make sure the download was complete.
    for recording in recordings:
        meeting_id = recording['meeting_id']
        recording_id = recording['id']
        print(f'deleting {meeting_id} :: {recording_id}')

        result = requests.delete(
            f'https://api.zoom.us/v2/meetings/{meeting_id}/recordings/{recording_id}',
            headers=headers,
        )


def process_meetings(headers, meetings, delete):
    if not os.path.exists('recordings'):
        os.mkdir('recordings')

    for meeting in meetings:
        recordings = meeting['recording_files']
        get_recordings(recordings)

        if delete:
            delete_recordings(recordings, headers)

def main(args):

    if 'ZOOM_JWT_TOKEN' in os.environ:
        token = os.getenv('ZOOM_JWT_TOKEN')
    elif args.token and os.path.exists(args.token):
        with open (args.token, 'r') as token_file:
            token = token_file.read().replace('\n', '')
    else:
        token = args.token

    if not token:
        exit('error: missing JWT token')

    headers = make_headers(token)
    months = make_months(args.start, args.end)

    meetings = get_meetings(headers, months)
    process_meetings(headers, meetings, args.delete)

    print('done!')

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description='Backs up Zoom cloud recordings')
    parser.add_argument(
        '--start',
        metavar='YYYY-MM-DD',
        required=True,
        help='first of month to start gathering recording meetings'
    )
    parser.add_argument(
        '--end',
        metavar='YYYY-MM-DD',
        required=True,
        help='first of month to end gathering recording meetings'
    )
    parser.add_argument(
        '--token',
        metavar='YOUR_JWT',
        required=False,
        help='the JWT you generate from your Zoom app, also can use env variable ZOOM_JWT_TOKEN or file path where token is placed'
    )
    parser.add_argument(
        '--delete',
        action='store_true',
        help='delete recordings after downloading them'
    )
    args = parser.parse_args()
    main(args)
