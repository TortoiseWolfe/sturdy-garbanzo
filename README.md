# sturdy-garbanzo
## Autogen
```bash
# Navigate to your workspace or desired directory
cd /path/to/your/workspace

# Create the project root directory
mkdir -p project-root-directory

# Change directory to the project root directory
cd project-root-directory

# Create the necessary sub-directories
mkdir -p docker autogen/templates autogen/definitions autogen/output src bin tests scripts lib docs

# Create a blank Dockerfile in the docker directory
touch docker/Dockerfile

# Create a blank docker-compose.yml file in the project root directory
touch docker-compose.yml

# Populate the Dockerfile with the necessary content
echo -e "FROM ubuntu:latest\n\
RUN apt-get update && apt-get install -y autogen build-essential\n\
COPY autogen /autogen\n\
WORKDIR /autogen\n\
RUN autogen -T templates/template.tpl definitions/definitions.def\n\
# (Continue with the rest of your build process, e.g., compile the generated code)" > docker/Dockerfile

# Populate the docker-compose.yml file with the necessary content
echo -e "version: '3'\n\
services:\n\
  autogen-service:\n\
    build:\n\
      context: ./docker\n\
      dockerfile: Dockerfile\n\
    volumes:\n\
      - ./autogen:/autogen\n\
      - ./src:/src\n\
    command: /bin/bash -c \"autogen -T autogen/templates/template.tpl autogen/definitions/definitions.def && (continue with your build process)\"" > docker-compose.yml
```

