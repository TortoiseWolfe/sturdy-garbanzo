# sturdy-garbanzo
https://youtu.be/V2qZ_lgxTzg?si=jQae-X3ZFy93iAyX
## Autogen
```bash
# Create the necessary sub-directories
mkdir -p docker autogen/templates autogen/definitions autogen/output src bin tests scripts lib docs

# Create a blank Dockerfile in the docker directory and populate it with the necessary content
echo -e "FROM continuumio/anaconda3:latest\n\
RUN apt-get update && apt-get install -y autogen build-essential\n\
COPY autogen /autogen\n\
WORKDIR /autogen\n\
RUN autogen -T templates/template.tpl definitions/definitions.def" > docker/Dockerfile

# Create a basic AutoGen template file
echo 'Hello, World!\nThis is a test of AutoGen with the name: %(name)s' > autogen/templates/template.tpl

# Create a basic AutoGen definition file
echo 'name = "AutoGen User";' > autogen/definitions/definitions.def

echo """
import autogen

def get_llm_config():
    config_list = [
        {
            'model': 'gpt-4',
            'api_key': 'API_KEY'
        }
    ]
    llm_config = {
        'request_timeout': 600,
        'seed': 42,
        'config_list': config_list,
        'temperature': 0
    }
    return llm_config

def create_assistant_agent():
    llm_config = get_llm_config()
    assistant = autogen.AssistantAgent(
        name='CTO',
        llm_config=llm_config,
        system_message='Chief technical officer of a tech company'
    )
    return assistant

def create_user_proxy_agent():
    llm_config = get_llm_config()
    user_proxy = autogen.UserProxyAgent(
        name='user_proxy',
        human_input_mode='NEVER',
        max_consecutive_auto_reply=10,
        is_termination_msg=lambda x: x.get('content', '').rstrip().endswith('TERMINATE'),
        code_execution_config={'work_dir': 'web'},
        llm_config=llm_config,
        system_message='Reply TERMINATE if the task has been solved at full satisfaction. Otherwise, reply CONTINUE, or the reason why the task is not solved yet.'
    )
    return user_proxy

def main():
    assistant = create_assistant_agent()
    user_proxy = create_user_proxy_agent()
    
    task1 = '''
    Write python code to output numbers 1 to 100, and then store the code in a file
    '''
    user_proxy.initiate_chat(assistant, message=task1)
    
    task2 = '''
    Change the code in the file you just created to instead output numbers 1 to 200
    '''
    user_proxy.initiate_chat(assistant, message=task2)

if __name__ == '__main__':
    main()
""" > src/app.py



# Create a blank docker-compose.yml file in the project root directory and populate it with the necessary content
echo -e "version: '3'\n\
services:\n\
  anaconda:\n\
    image: continuumio/anaconda3\n\
    build:\n\
      context: .  # Updated context to project root directory\n\
      dockerfile: docker/Dockerfile  # Updated path to Dockerfile\n\
    stdin_open: true\n\
    tty: true\n\
    command: /bin/bash\n\
    volumes:\n\
      - ./autogen:/autogen\n\
      - ./src:/src" > docker-compose.yml

```
```bash
docker-compose run anaconda
```

```bash
# Create new conda environment
conda create -n agen python=3.11.4
# Activate the new environment
conda activate agen
# Install AutoGen
pip install pyautogen
# check version
python --version
```

Yes, there are several Docker images available for setting up an Anaconda-based development environment. Here are some options:

1. **Official Anaconda Docker Image:**
   - There's an official Docker image provided by Continuum Analytics on Docker Hub. This image has Anaconda pre-installed and ready to use, with the Anaconda distribution installed in the `/opt/conda` folder, ensuring the `conda` command is available in the default user's path【18†(source)】.

2. **Anaconda and Miniconda Docker Images by Anaconda, Inc.:**
   - Anaconda, Inc. provides both Anaconda and Miniconda Docker images which can be found in their documentation. These images can serve as a base for your development environment, especially if you are integrating with other tools or frameworks【19†(source)】.

3. **Anaconda Development Container Images by Microsoft:**
   - Microsoft also provides pre-built Anaconda Development Container Images on Docker Hub. You can reference these images directly in your Docker configuration files, or update the `FROM` statement in your own Dockerfile to use these images as a base. An example Dockerfile is included in the repository, which might be helpful as a reference【20†(source)】.

4. **Open Source Version of Anaconda on Docker Hub:**
   - There is also an open-source version of Anaconda available on Docker Hub provided by Continuum Analytics, Inc. This version includes over 100 of the most popular Python packages for data science and provides access to over 720 Python and R packages that can be easily installed using the `conda` dependency manager【21†(source)】.

```bash
# Create a blank docker-compose.yml file in the project root directory and populate it with the necessary content
echo -e "version: '3'\n\
services:\n\
  autogen-service:\n\
    build:\n\
      context: .  # Updated context to project root directory\n\
      dockerfile: docker/Dockerfile  # Updated path to Dockerfile\n\
    volumes:\n\
      - ./autogen:/autogen\n\
      - ./src:/src\n\
    command: /bin/bash -c \"autogen -T autogen/templates/template.tpl autogen/definitions/definitions.def\"" > docker-compose.yml
```
You can choose any of these images based on your project requirements and the level of customization or control you need over the environment. Once you've chosen an image, you can use it as the base image in your Dockerfile, and then add any additional setup or configuration needed for your project.