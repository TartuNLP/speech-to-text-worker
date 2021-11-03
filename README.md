# Automatic Speech Recognition (ASR) Worker

A component that automatically recognises speech from an audio file and transcribes it into text.

## Setup

The worker can be used by running the prebuilt [docker image](ghcr.io/project-mtee/asr-worker). The 
container is designed to run in a CPU environment. For a manual setup, please refer to the included Dockerfile and 
the pip packages specification described in `requirements.txt`. 

The worker depends on the following components:
- [RabbitMQ message broker](https://www.rabbitmq.com/)

The following environment variables should be specified when running the container:
- `MQ_USERNAME` - RabbitMQ username
- `MQ_PASSWORD` - RabbitMQ user password
- `MQ_HOST` - RabbitMQ host
- `MQ_PORT` (optional) - RabbitMQ port (`5672` by default)
- `HTTP_HOST` - speech recognition service API HOST
- `HTTP_USERNAME` - speech recognition service API username (`user` by default)
- `HTTP_PASSWORD` - speech recognition service API password (`pass` by default)

### Performance and Hardware Requirements

TODO

### Request Format

The worker consumes speech recognition requests from a RabbitMQ message broker and responds with the transcribed text 
straight to the ASR service. 

Requests should be published with the following parameters:
- Exchange name: `speech-to-text` (exchange type is `direct`)
- Routing key: `speech-to-text.<src>` where `<src>` refers to 2-letter ISO language code of the given text. For 
  example `speech-to-text.et`
- Message properties:
  - Correlation ID - a UID for each request that can be used to correlate requests and responses.
- JSON-formatted message content with the following keys:
  - `correlation_id` – same as the message property correlation ID
  - `file_extension` – the file extension of the uploaded audio file (.wav, .mp3, etc.)

The worker will return a response with the following parameters:
- JSON-formatted message content with the following keys:
  - `success` – whether the transcription of the audio file was a success or not (boolean).
  - `result` – the transcribed text.