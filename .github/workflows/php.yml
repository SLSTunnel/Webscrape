import os
import datetime
import dateutil.relativedelta
from googleapiclient.discovery import build
from googleapiclient.errors import HttpError
from googleapiclient.http import MediaFileUpload
from google.oauth2 import service_account
import moviepy.editor as mp
import shutil

# Set up YouTube API credentials
SERVICE_ACCOUNT_FILE = 'path/to/service_account_key.json'
SCOPES = ['https://www.googleapis.com/auth/youtube.force-ssl']
creds = service_account.Credentials.from_service_account_file(
        SERVICE_ACCOUNT_FILE, SCOPES)
youtube = build('youtube', 'v3', credentials=creds)

# Set up accounts to scrape from
accounts = ['account1', 'account2', 'account3']

# Set up output folder
output_folder = './videos'

# Set up compilation video settings
total_vid_length = 10 * 60  # 10 minutes

def scrape_videos(username, password, days):
    # Scrape videos from account
    looter = InstagramLooter()
    if not looter.logged_in():
        looter.login(username, password)
    today = datetime.date.today()
    timeframe = (today, today - dateutil.relativedelta.relativedelta(days=days))
    num_downloaded = looter.download(output_folder, media_count=30, timeframe=timeframe)
    print("Downloaded " + str(num_downloaded) + " videos successfully")
    print("")

def make_compilation(path='./', total_vid_length=10*60):
    # Create a compilation video from downloaded videos
    videos = [os.path.join(path, f) for f in os.listdir(path) if f.endswith('.mp4')]
    clips = [mp.VideoFileClip(v) for v in videos]
    final_clip = mp.concatenate_videoclips(clips)
    final_clip.write_videofile('output.mp4')

def upload_to_youtube(video_file, title, description):
    # Upload compilation video to YouTube
    request_body = {
        'snippet': {
            'title': title,
            'description': description,
            'tags': ['dank memes', 'compilation']
        },
        'status': {
            'privacyStatus': 'public'
        }
    }
    media_file = MediaFileUpload(video_file, chunksize=-1, resumable=True)
    response_upload = youtube.videos().insert(
        part='snippet,status',
        body=request_body,
        media_body=media_file
    ).execute()
    print("Upload Successful!")

def cleanup(video_directory, output_file):
    # Remove temporary files
    shutil.rmtree(video_directory, ignore_errors=True)
    try:
        os.remove(output_file)
    except OSError as e:
        print("Error: %s - %s." % (e.filename, e.strerror))
    print("Removed temp files!")

def routine():
    now = datetime.datetime.now()
    print(now.year, now.month, now.day, now.hour, now.minute, now.second)
    title = "TRY NOT TO LAUGH (BEST Dank video memes) V1"
    video_directory = "./Memes_" + str(now.month).upper() + "_" + str(now.year) + "_V" + str(now.day) + "/"
    output_file = "./" + str(now.month).upper() + "_" + str(now.year) + "_v" + str(now.day) + ".mp4"
    if not os.path.exists(video_directory):
        os.makedirs(video_directory)
    for account in accounts:
        scrape_videos(account, 'password', 1)
    make_compilation(video_directory, total_vid_length)
    upload_to_youtube(output_file, title, "")
    cleanup(video_directory, output_file)

if name == "__main__":
    routine()
