# Performance testing in the WebRTC arena

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.14731689.svg)](https://doi.org/10.5281/zenodo.14731689)

Reproduction package for the paper "Performance testing in the WebRTC arena: a case study of three real-time communication systems". This description contains detailed steps to reproduce the results on the paper.

The almost complete reproduction package can be found in Zenodo ([https://doi.org/10.5281/zenodo.14731689](https://doi.org/10.5281/zenodo.14731689)) and contains the following files:

```
.
├── dataset-combined.zip            # Raw data results of the paper ready for analysis (~17 GB uncompressed)
├── openvidu-loadtest-3.0.0.zip     # OpenVidu LoadTest code
├── mediafiles.zip                  # Media files used in the paper for the users in OpenVidu LoadTest
├── analysis.ipynb                  # Jupyter Notebook for data analysis
├── extract.ipynb                   # Jupyter Notebook for extraction of data from an ELK stack
├── .env.template                   # Template of .env file used for extraction of data
└── README.md                       # This file
```

Note that the raw video recordings for the paper can't be uploaded to Zenodo as the total size of the recordings is ~500GB. If you need these files, you can contact the authors of this paper at [ivan.chicano@urjc.es](mailto:ivan.chicano@urjc.es).

## Index
- [Reproducing the experiments of the paper](#reproducing-the-experiments-of-the-paper)
    - [Step 1. Setup](#step-1-setup)
    - [Step 2. Configuring and running OVLT](#step-2-configuring-and-running-ovlt)
    - [Step 3. Running QoE Analysis](#step-3-running-qoe-analysis)
    - [Step 4. Results and analysis](#step-4-results-and-analysis)

## Reproducing the experiments of the paper

### Step 1. Setup

For the paper AWS EC2 instances were used, so an AWS account is needed, with permissions to create and terminate EC2 instances and create AMIs.

OVLT also needs an ElasticSearch and Kibana stack (ELK stack from now on) for it to save relevant data, instructions for setting up version 7.8 (version used in the paper) can be found [here](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/setup.html).

To save the recordings and relevant stats files OVLT needs AWS S3 or MinIO.

You can find instructions for configuring and using the OpenVidu LoadTest (OVLT) version used in the paper in the README.md file inside `openvidu-loadtest-3.0.0.zip` (unzip the tool before using). OVLT needs Java 11+ and Maven 3+ installed.

Instructions for deploying OpenVidu 2 with Kurento and Mediasoup (version 2.30.0 was used in the paper) can be found [here](https://docs.openvidu.io/en/2.30.0/deployment/enterprise/aws/). Remove the limits on bandwidth by setting the following configuration parameters to 0:

- OPENVIDU_STREAMS_VIDEO_MAX_RECV_BANDWIDTH
- OPENVIDU_STREAMS_VIDEO_MIN_RECV_BANDWIDTH
- OPENVIDU_STREAMS_VIDEO_MAX_SEND_BANDWIDTH
- OPENVIDU_STREAMS_VIDEO_MIN_SEND_BANDWIDTH.

Information on how to do this can be found [here](https://docs.openvidu.io/en/2.30.0/reference-docs/openvidu-config/).

Instructions for deploying LiveKit with Pion can be found [here](https://docs.livekit.io/home/self-hosting/deployment/) (version 1.6.1 of LiveKit was used in the paper, can be found [here](https://github.com/livekit/livekit/releases/tag/v1.6.1))
First, you will need to configure the tool to run against an OpenVidu or LiveKit deployment, as well as configuring the connections to the ELK stack
For more detail explanations on configuring OVLT, see the `README.md` Markdown file in

In short, you will need:
    - An OpenVidu or LiveKit deployment
    - OpenVidu LoadTest
    - An ELK stack
    - AWS access to EC2, AMI creation and S3 (or MinIO as an alternative)

### Step 2. Configuring and running OVLT

First unzip `openvidu-loadtest-3.0.0.zip`. The directory where the project is unzipped will be known as root directory.

You will need to create the AWS AMI and a Security Group for the User Emulators to create the EC2 instances (also known as Browser Emulators). For this, refer to the `README.md` file in the root directory of OVLT, section Usage instructions > Launch workers > For testing on AWS

After this, you can configure the controller. More information about configuring the controller can be found in the README.md, section Usage instructions > Execute controller.

First, we will need to edit the `loadtest-controller/src/main/resources/application.properties` file. The explanation for each section of the configuration are the following:

#### Load Test Parameters
If testing against an OpenVidu deployment fill OPENVIDU_URL and OPENVIDU_SECRET with your deployment's URL and Secret key.

If testing against a LiveKit deployment, fill OPENVIDU_URL with your deployment URL, and remove the OPENVIDU_SECRET configuration key and add the following configuration keys with your LiveKit API Key and Secret.

```
LIVEKIT_API_KEY=
LIVEKIT_API_SECRET=
```

The following configuration was used for the paper:

```
SESSION_NAME_PREFIX=LoadTestSession
USER_NAME_PREFIX=User
SECONDS_TO_WAIT_BETWEEN_PARTICIPANTS=5
SECONDS_TO_WAIT_BETWEEN_SESSIONS=0
SECONDS_TO_WAIT_BEFORE_TEST_FINISHED=0
SECONDS_TO_WAIT_BETWEEN_TEST_CASES=0
```

#### ELK Monitoring Parameters

Configure this section with your ELK deployment parameters.

#### For testing locally

Leave empty.

#### For testing with AWS
Fill AWS_ACCESS_KEY and AWS_SECRET_ACCESS_KEY with your AWS account's access key and secret access key.

Fill WORKER_AMI_ID with the AMI ID of the AMI we have generated for the User Emulators, and WORKER_SECURITY_GROUP_ID with the security group you created.
The following instance type, region and availability zone was used in the paper:

```
WORKER_INSTANCE_TYPE=c5.xlarge
WORKER_INSTANCE_REGION=us-east-1
WORKER_AVAILABILITY_ZONE=us-east-1f
```

For the paper, WORKERS_NUMBER_AT_THE_BEGINNING was filled with the expected number of users that test would hold to keep the tests as short as possible, and WORKERS_RAMP_UP was filled with half the previous value.

#### For AUTOMATIC distribution participants to workers

Keep as is.

#### For MANUAL distribution participants to workers

The following parameters were used for the paper:

```
MANUAL_PARTICIPANTS_ALLOCATION=true
USERS_PER_WORKER=1
```

#### For VIDEO QUALITY control

The following parameters were used for the paper:

```
MEDIANODE_LOAD_FOR_START_RECORDING=0
RECORDING_SESSION_GRUPED_BY=0
VIDEO_PADDING_DURATION=1
VIDEO_FRAGMENT_DURATION=15
QOE_ANALYSIS_RECORDINGS=true
RECORDING_WORKERS_AT_THE_BEGINNING=0
QOE_ANALYSIS_IN_SITU=false
```

Choose a name for a bucket if using S3 in S3_BUCKET_NAME. Recommendation: Use different buckets for different tests to avoid files being overwritten.

#### For choosing the video to use

As of the date of publishing this package, the media files used by the User Emulators should be automatically downloaded if using the following configuration (the configuration used in the paper):

```
VIDEO_TYPE=INTERVIEW
VIDEO_WIDTH=640
VIDEO_HEIGHT=480
VIDEO_FPS=30
```

If the files no longer exist remotely, they can be found in the `mediafiles.zip` in the package. You will need to upload them to a platform accessible to the browser emulator for download, and use the following configuration:

```
VIDEO_TYPE=CUSTOM
VIDEO_URL=The URL of the video file
AUDIO_URL=The URL of the audio file
```

#### MinIO credentials and information if using it to save recordings instead of S3

Only fill if using MinIO, else leave empty.

#### For retrying the participant creation when it fails

The following parameters were used for the paper:

```
RETRY_MODE=true
RETRY_TIMES=5
```

#### Miscelaneous configuration

The following parameters were used for the paper:

```
BATCHES=true
WAIT_COMPLETE=true
DEBUG_VNC=false
```

In the following table you can find the values used for BATCHES_MAX_REQUESTS in each test.

| Media Server | Session Configuration                | BATCHES_MAX_REQUESTS |
|--------------|--------------------------------------|----------------------|
| Kurento      | 2 publishers per session             | 5                    |
| Kurento      | 8 publishers per session             | 5                    |
| Kurento      | 3 publishers and 10 subscribers per session | 5                    |
| Kurento      | 3 publishers and 40 subscribers per session | 5                    |
| Mediasoup    | 2 publishers per session             | 20                   |
| Mediasoup    | 8 publishers per session             | 20                   |
| Mediasoup    | 3 publishers and 10 subscribers per session | 20                   |
| Mediasoup    | 3 publishers and 40 subscribers per session | 20                   |
| Pion         | 2 publishers per session             | 10                   |
| Pion         | 8 publishers per session             | 10                   |
| Pion         | 3 publishers and 10 subscribers per session | 10                   |
| Pion         | 3 publishers and 40 subscribers per session | 10                   |

After configuring the resources file, configure the `loadtest-controller/src/main/resources/test_cases.json` file with the test cases you want to run. Recommendation: use one `test_cases.json` file for each test. A detailed explanation can be found in README.md in section Configure session typology.

Of note, in the following table you can find the values used for startingParticipants for each test:

| Media Server | Session Configuration                | startingParticipants |
|--------------|--------------------------------------|----------------------|
| Kurento      | 2 publishers per session             | 10                   |
| Kurento      | 8 publishers per session             | 10                   |
| Kurento      | 3 publishers and 10 subscribers per session | 20                   |
| Kurento      | 3 publishers and 40 subscribers per session | 20                   |
| Mediasoup    | 2 publishers per session             | 100                  |
| Mediasoup    | 8 publishers per session             | 50                   |
| Mediasoup    | 3 publishers and 10 subscribers per session | 100                  |
| Mediasoup    | 3 publishers and 40 subscribers per session | 100                  |
| Pion         | 2 publishers per session             | 100                  |
| Pion         | 8 publishers per session             | 50                   |
| Pion         | 3 publishers and 10 subscribers per session | 100                  |
| Pion         | 3 publishers and 40 subscribers per session | 100                  |

With everything set up, you can now run the tests with the following commands:

```bash
cd loadtest-controller
mvn spring-boot:run > test.log 2>&1
```

Note: The logs of the controller are used for analysis, so saving them is important (see the redirection in the command). We recommend changing the name of the log according to the test being run for easier tracking.

### Step 3. Running QoE Analysis

Warning: QoE Analysis can take a long time when analyzing all the resulting recordings from a test, even in the order of days or weeks. It is recommended to run the scripts on multiple machines or a big machine with a lot of CPUs.

After a successful run of a test, you can find in the S3/MinIO Bucket the following files and directories:

- stats: WebRTC stats recorded by the user emulators
- Multiple webm files: Recordings of the communication between users

With the recordings, we can now analyze the Quality of Experience (QoE) using Full-Reference models as described in the paper. The resources used for QoE analysis can be found in the OVLT tool, for this, from the root directory, use the command:

```bash
cd browser-emulator
```

We will need the following installed in the machine that will run the analysis:

- Node 16+
- Python 3.11 and pip3
- [VMAF v2.3.1](https://github.com/Netflix/vmaf/releases/tag/v2.3.1), set a VMAF_PATH environment variable with the path to the vmaf binary and move the `vmaf_v0.6.1.json` file found in the model directory to `/usr/local/share/vmaf/models/vmaf_v0.6.1.json`
- [ViSQOL v3.3.3](https://github.com/google/visqol/releases/tag/v3.3.3), set a VISQOL_PATH environment variable with the path to the directory that contains `bazel-bin/visqol`
- [Tesseract OCR v5.3.1 (any v5+ should work)](https://github.com/tesseract-ocr/tesseract), it is highly recommended to build it disabling multithreading as explained [here](https://tesseract-ocr.github.io/tessdoc/Compiling-%E2%80%93-GitInstallation.html#release-builds-for-mass-production) to improve performance.
    - You will probably need to save the necessary model found [here](https://github.com/tesseract-ocr/tessdata/raw/main/eng.traineddata) in `/usr/local/share/tessdata/eng.traineddata`
Install the node dependencies with:

```bash
npm install
```

Install the QoE analysis scripts' dependencies with:
```bash
pip3 install -r qoe_scripts/requirements.txt
```

Save the individual recordings you want to analyze in `./recordings/qoe` directory.

Unzip mediafiles.zip, resulting in 2 files:
- interview_480p_30fps.y4m: Reference video file
- interview.wav: Reference audio file

You will now need to edit the `qoe-results-processing-config.json` file found in the current directory, this is the configuration used in the paper:

```json
{
  "presenter_video_file_location": "absolute path to the reference video file",
  "presenter_audio_file_location": "absolute path to the reference audio file",
  "width": 640,
  "height": 480,
  "framerate": 30,
  "padding_duration": 1,
  "fragment_duration": 15
}
```

You can now run
```bash
npm run qoe -- --onlyfiles
```

Recommendation: Run using `nohup` and in the background to avoid the possibility of the script being interrupted.

```bash
nohup npm run qoe -- --onlyfiles > qoe_analysis.log 2>&1 &
```

After finishing, the resulting files can be found in the current directory as `v-*.json` (where * is part of the name of the recording). It is recommended to move them to a new directory for organization purposes. For example:

```bash
mkdir kurento-2p
mv v-*.json kurento-2p/
```

### Step 4. Results and data analysis

For the results' analysis, we have 2 Jupyter Notebooks. Both Notebooks have the results from the paper, but can be used to analyze other tests.

- `extract.ipynb`: For extraction of data from an ELK stack
- `analysis.ipynb`: For data analysis

To analyze new tests after running OVLT and doing the QoE Analysis, first we need to extract the relevant data from ELK for the analysis Jupyter Notebook. For this, we will use the `extract.ipynb` Notebook.

- First, edit the `.env.template` file, set the location of your ELK stack and change the file name to `.env`.
- Next, create a directory `dfs_final` in the directory where both Notebooks reside:
```bash
mkdir dfs_final
```
- You will need to change the variables on the third cell to the specific index name and test times for your tests.
    - The first index_list_names contains the index name in ELK for the first test to analyze, and its duration from start (min) to finish (max) should be in the first position of test times.

Run the Notebook and the CSV data files from ELK by OVLT will be extracted.

For the analysis Notebook, you will need to edit the second cell variables using the same values you used in the extract Notebook.

These are the necessary directories where the data files must be:
- `dfs_final`: Extracted files from ELK
- `stats`: WebRTC stats files from OVLT, as saved in S3/MinIO, should be saved for each test in directory stats/[index_name]/, where [index_name] is a name in the index_list_names array
- `logs`: Log files from OVLT Controller, saved in logs/[index_name].log for each test
- `qoe`: Results files of QoE Analysis, saved in qoe/[index_name]/ for each test

Now you can run the Notebook to obtain all plots and tables seen in the paper and some more. These are also saved in the `plots` and `tables` directories.