

# How to Run Claude Code with local Gemma4:26b

## Introduction:
This HOWTO provides a step-by-step guide to running Claude Code with Gemma4 served locally via Ollama.

With a modest hardware configuration (details in the next section), I can get Claude working at a reasonable speed. Each step took approximately 4 minutes.

For small task, like vibe-coding a simple RAG script with Python, it gets the job done. It's nowhere near the performance of a commercial API, but if you’ve got the curiosity, it’s a fun project to try. If not, feel free to skip this one.

Bon, On y va!

## Hardware Setup
I used a fairly modest PC to serve as an LLM server (to run Ollama with Gemma4:26b):
- A Debian Linux box, Intel(R) Core(TM) i7-10700 CPU @ 2.90GHz, 512 GB SSD, 64GB RAM. NVIDIA GeForce GTX 1660 with 6GB RAM, CUDA version 12.4.

- A MacBook Pro M1 laptop with 16GB RAM (or whatever PC you have) to run Claude Code.

- Both machines are on the same VLAN. I set a manual IP address, 192.168.1.101, for the Debian box.

## Software Setup

### Server side:
- Install Ollama:
  ```bash
  $ curl -fsSL https://ollama.com/install.sh | sh
  ```

- Configure Ollama as a local API server:
  ```bash
  $ sudo systemctl edit ollama
  ```
  Add this configuration to the file and then save it:
  ```
  [Service]
  Environment="OLLAMA_MODELS=/PATH_TO/.ollama/models"
  Environment="OLLAMA_FLASH_ATTENTION=1"  #(optional)
  Environment="OLLAMA_KEEP_ALIVE=-1"
  Environment="OLLAMA_HOST=0.0.0.0"
  Environment="OLLAMA_ORIGINS=*"
  Environment="OLLAMA_CONTEXT_LENGTH=65536"
  Environment="OLLAMA_NUM_PARALLEL=1"    #(optional)
  ```
  Note:
    1. Replace PATH_TO with the path where you want to save models.
    2. You may have to remove the optional lines if you do not have a GPU/CUDA; otherwise, you might encounter an error.
    3. Reduce OLLAMA_CONTEXT_LENGTH to 32, 16K, or 8K if your box doesn't have enough memory.


- Pull Gemma4:26b:
  ```bash
  $ ollama pull gemma4:26b
  ```
- Verify if it is ready:
  ```bash
  $ ollama list  
  NAME                       ID              SIZE      MODIFIED        
  gemma4:26b                 5571076f3d70    17 GB     43 hours ago  
  ```
- Create a 64K input context length:

  - First, create a Modelfile (save it anywhere):
    ```
    FROM gemma4:21b
    PARAMETER num_ctx 65536
    PARAMETER temperature 1.0
    PARAMETER top_p 0.95
    PARAMETER top_k 64
    ```
    This configuration uses your existing gemma4:26b as the base.
    Set the context window to your desired size (e.g., 32768 for 32k context).
    Adjust this value based on your available VRAM.

  - In the same folder as the Modelfile, run:
    ```bash
    $ ollama create gemma4-64k -f ./Modelfile
    ```
  - Verify it:
    ```bash
    $ ollama list
    NAME                       ID              SIZE      MODIFIED     
    gemma4-64k:latest          62a18541dae0    17 GB     20 hours ago    
    gemma4:26b                 5571076f3d70    17 GB     43 hours ago  
    ```

- Apply the changes:
  ```bash
  $ sudo systemctl daemon-reload
  $ sudo systemctl restart ollama
  ```
  
  The server is ready to serve.

### Client side (MacBook Pro):
- Install Ollama:
  ```bash
  $ curl -fsSL https://ollama.com/install.sh | sh
  ```

- Export environment variables:
  ```bash
  $ export ANTHROPIC_BASE_URL="192.168.1.101"
  $ export ANTHROPIC_AUTH_TOKEN="ollama"
  $ export ANTHROPIC_TIMEOUT=600000
  ```
  It is important to adjust the TIMEOUT depending on how powerful your system is.

### Ready to test:
```bash
$ claude --model gemma4-64k:latest
```
The first run will take quite a long time. Please be patient!

Enjoy it.
