# README.md

## Overview
This project utilizes various libraries, including SpaCy, Geonamescache, and Instagrapi, to process text data with a specific focus on hashtags and graffiti-related content. The goal is to parse hashtags into meaningful words, identify entities, and classify terms related to graffiti, cities, and railroad lingo.

## Features
### 1. Initialization and Setup
- **Geonamescache** is used for city and country information.
- **Instagrapi** handles interactions with Instagram data.
- **SpaCy** is used for natural language processing. Custom extensions are added to SpaCy's `Token` class to include properties like `is_city`, `is_graffiti_lingo`, and `is_railroad_lingo`.

### 2. Wordlist Initialization
The `initialize_words` function loads wordlists from text files for:
- General vocabulary
- City names
- Graffiti lingo
- Railroad lingo

These wordlists are essential for parsing and classifying hashtags.

### 3. Parsing Hashtags
The pipeline processes hashtags using the following components:

#### **`mention_hashtags`**
Merges tokens starting with `#` or `@` into single tokens.

#### **`mention_hashtags_set_extension`**
Adds extensions to tokens, identifying if they are hashtags or mentions.

#### **`hashtag_splitter_tagger`**
Splits hashtags into meaningful words using a recursive approach.

#### **`graffiti_entities_lookup`**
Identifies graffiti-related entities like writers and crews, and looks for specific patterns or keywords in hashtags.

### 4. Entity Identification Examples

#### **How Hashtags are Split into Words and Entities**
For example, the hashtag `#freightgraffitiChicago` is processed as follows:
1. The hashtag is stripped of the `#` symbol.
2. Words are identified sequentially from the beginning:
   - `freight`
   - `graffiti`
   - `Chicago`
3. Each word is checked against predefined wordlists:
   - `freight` matches general vocabulary.
   - `graffiti` matches graffiti lingo.
   - `Chicago` is identified as a city using Geonamescache.

#### **Identifying Graffiti Writers and Crews**
- Example: `#mecro`
  - The term `mecro` is identified as an out-of-vocabulary word (OOV).
  - If it is not part of the wordlist, it is classified as a graffiti writer.

- Example: `#mskcrew`
  - The term `msk` is identified, and the suffix `crew` signifies it as a graffiti crew.

#### **Common Hashtags in the Graffiti Community**
- `#blackbook`: Refers to sketchbooks used by graffiti artists to draft designs.
- `#burnersonthestreet`: Highlights impressive or notable street graffiti.
- `#vandalsMexico`: Represents graffiti or street art associated with Mexico.

Each of these hashtags undergoes the same processing pipeline to extract meaningful words and classify entities.

## How it Works
### Recursive Parsing of Hashtags
The `parse_tag` function recursively identifies words within a hashtag:
1. The input string is split by dashes or processed as a whole.
2. Words are extracted iteratively from the start of the string.
3. Each word is matched against the wordlist or categorized as an OOV entity.

### Setting Token Extensions
Custom extensions allow the program to annotate tokens with additional metadata, such as:
- `is_city`: Whether the token represents a city.
- `is_graffiti_lingo`: Whether the token is graffiti-related.
- `custom_entity_cat`: Custom classification for unidentified entities.

## Docker Integration
### Dockerfile Setup
The application can be containerized to create multiple instances using Docker. Below is the Dockerfile used for building the image:

```dockerfile
FROM python:3.11.0b4-buster

ARG _USER="spacy"
ARG _UID="1001"
ARG _GID="100"
ARG _SHELL="/bin/bash"

# Install apt dependencies
RUN apt-get update && apt-get install -y \
    nano \
    git \
    wget

RUN useradd -m -s "${_SHELL}" -N -u "${_UID}" "${_USER}"

ENV USER ${_USER}
ENV UID ${_UID}
ENV GID ${_GID}
ENV HOME /home/${_USER}
ENV PATH "${HOME}/.local/bin/:${PATH}"
ENV PIP_NO_CACHE_DIR "true"

RUN mkdir /home/${_USER}/app && chown ${UID}:${GID} /home/${_USER}/app

USER ${_USER}

COPY --chown=${UID}:${GID} config* /home/${_USER}/app
COPY --chown=${UID}:${GID} requirements* /home/${_USER}/app
COPY --chown=${UID}:${GID} ./* /home/${_USER}/app/

WORKDIR /home/${_USER}/app

RUN pip install -r requirements.txt

CMD bash
```

### Instructions for Running with Docker
#### Clone the Project
First, clone the repository:
```bash
git clone https://github.com/abundis-rmn2/Spacy-Hashtag-Geolocator.git
```

#### Build the Docker Image
Run the following command to build the Docker image with the name `hashtags` (you can use any name you prefer):
```bash
docker build -t hashtags .
```

#### Create and Run a Container
To create and run a container, use the following command. In this case, we name the container `hash1`:
```bash
docker run -it --name hash1 -d hashtags
```

#### Optional: Use `screen` to Create a Session
If you want to keep the process running in a separate terminal session, you can use `screen`:
```bash
screen -S hash
```

#### Execute Commands Inside the Container
Run a specific script or test inside the container by specifying the container name and the MUID:
```bash
# docker exec -t [container_name] python test.py -MUID=[MUID]
docker exec -t hash1 python test.py -MUID=nearaxs_1_hashtagTop_9_48c69711
```

#### Example Output
After executing the process, you will see output indicating the script has started:
```
Initialize Words wl file
<class 'list'>
185606
Initialize Words cities file
<class 'list'>
26797
Initialize Words graffiti-lingo file
<class 'list'>
551
Initialize Words railroad-lingo file
<class 'list'>
681
Looking for caption in MUID: fr8porn_1_hashtagTop_9_3bf76f18
MUID found : 513
```

### Benefits of Using Docker
- **Scalability**: Run multiple instances of the application by creating additional containers.
- **Consistency**: The Docker image ensures the environment remains consistent across deployments.
- **Ease of Use**: The containerized setup simplifies the process of setting up and running the application.

## Conclusion
This project provides a structured way to analyze hashtags, focusing on graffiti-related content. It identifies cities, graffiti terms, and unique entities like writers and crews, enhancing the understanding of social media data in this niche.
