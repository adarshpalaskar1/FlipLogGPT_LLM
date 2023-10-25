# FlipLogGPT
Interactive LLM for logs and security analysis with vectorstores

Built with [LangChain](https://github.com/hwchase17/langchain), [GPT4All](https://github.com/nomic-ai/gpt4all), [Chroma](https://www.trychroma.com/) and [SentenceTransformers](https://www.sbert.net/).

# Presentation
[Presentation Link](https://drive.google.com/file/d/1ywUDUhdxkC7a2chzTz34hlQ68A8GiIOS/view?usp=sharing)  

# Block Diagram of the approach
![fliploggpt](https://github.com/adarshpalaskar1/FlipLogGPT_LLM/assets/83298237/cc8b4935-08c9-425a-b8e3-aa7c0840e414)

# Environment Setup
In order to set your environment up to run the code here, first install all requirements:

```shell
pip3 install -r requirements.txt
```

Then, download the LLM model and place it in a directory of your choice:
- LLM: default to [ggml-gpt4all-j-v1.3-groovy.bin](https://gpt4all.io/models/ggml-gpt4all-j-v1.3-groovy.bin). If you prefer a different GPT4All-J compatible model, just download it and reference it in your `.env` file.

Copy the `example.env` template into `.env`
```shell
cp example.env .env
```

and edit the variables appropriately in the `.env` file.
```
MODEL_TYPE: supports LlamaCpp or GPT4All
PERSIST_DIRECTORY: is the folder you want your vectorstore in
MODEL_PATH: Path to your GPT4All or LlamaCpp supported LLM
MODEL_N_CTX: Maximum token limit for the LLM model
MODEL_N_BATCH: Number of tokens in the prompt that are fed into the model at a time. Optimal value differs a lot depending on the model (8 works well for GPT4All, and 1024 is better for LlamaCpp)
EMBEDDINGS_MODEL_NAME: SentenceTransformers embeddings model name (see https://www.sbert.net/docs/pretrained_models.html)
TARGET_SOURCE_CHUNKS: The amount of chunks (sources) that will be used to answer a question
```

Note: because of the way `langchain` loads the `SentenceTransformers` embeddings, the first time you run the script it will require internet connection to download the embeddings model itself.

## Dataset
https://anonymous.4open.science/r/chatgpt-for-logparsing/dataset/Apache/Apache_2k.log

## Instructions for ingesting your own dataset

Put any and all your files into the `source_documents` directory

The supported extensions are:

   - `.csv`: CSV,
   - `.docx`: Word Document,
   - `.doc`: Word Document,
   - `.enex`: EverNote,
   - `.eml`: Email,
   - `.epub`: EPub,
   - `.html`: HTML File,
   - `.md`: Markdown,
   - `.msg`: Outlook Message,
   - `.odt`: Open Document Text,
   - `.pdf`: Portable Document Format (PDF),
   - `.pptx` : PowerPoint Document,
   - `.ppt` : PowerPoint Document,
   - `.txt`: Text file (UTF-8),

Run the following command to ingest all the data.

```shell
python ingest.py
```

Output should look like this:

```shell
Creating new vectorstore
Loading documents from source_documents
Loading new documents: 100%|██████████████████████| 1/1 [00:01<00:00,  1.73s/it]
Loaded 1 new documents from source_documents
Split into 90 chunks of text (max. 500 tokens each)
Creating embeddings. May take some minutes...
Using embedded DuckDB with persistence: data will be stored in: db
Ingestion complete! You can now run FlipLogGPT.py to query your documents
```

It will create a `db` folder containing the local vectorstore. Will take 20-30 seconds per document, depending on the size of the document.
You can ingest as many documents as you want, and all will be accumulated in the local embeddings database.
If you want to start from an empty database, delete the `db` folder.

Note: during the ingest process no data leaves your local environment. You could ingest without an internet connection, except for the first time you run the ingest script, when the embeddings model is downloaded.

## To run your query use:

```shell
python FlipLogGPT.py
```

And wait for the script to require your input.

```plaintext
> Enter a query:
```

Hit enter. You'll need to wait 20-30 seconds (depending on your machine) while the LLM model consumes the prompt and prepares the answer. Once done, it will print the answer and the 4 sources it used as context from your documents; you can then ask another question without re-running the script, just wait for the prompt again.

Note: you could turn off your internet connection, and the script inference would still work. No data gets out of your local environment.

Type `exit` to finish the script.


# How does it work?
Selecting the right local models and the power of `LangChain` you can run the entire pipeline locally, without any data leaving your environment, and with reasonable performance.

- `ingest.py` uses `LangChain` tools to parse the document and create embeddings locally using `HuggingFaceEmbeddings` (`SentenceTransformers`). It then stores the result in a local vector database using `Chroma` vector store.
- `FlipLogGPT.py` uses a local LLM based on `GPT4All-J` or `LlamaCpp` to understand questions and create answers. The context for the answers is extracted from the local vector store using a similarity search to locate the right piece of context from the docs.
- `GPT4All-J` wrapper was introduced in LangChain 0.0.162.
