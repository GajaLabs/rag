# RAG Document Chatbot - Complete Documentation

## 1. Project overview

RAG Document Chatbot is a Streamlit web application that lets a user upload a
PDF and ask questions about its contents.

RAG means **Retrieval-Augmented Generation**:

1. The application retrieves the most relevant parts of the uploaded document.
2. It sends only those parts, together with the question, to a language model.
3. The language model generates an answer grounded in the retrieved document
   context.

The application does not send the entire PDF directly to the language model.
Instead, it uses local text extraction, local embeddings, and a local FAISS
vector index before making the Groq API request.

## 2. Repository map

```text
RAG-DOUCMENT-CHATBOT/
|
|-- app.py                 Main Streamlit application and RAG pipeline
|-- requirements.txt       Python packages installed locally and in the cloud
|-- .env.example            Safe template for local environment variables
|-- .env                    Local secrets; ignored by Git and never committed
|-- .gitignore              Files and folders excluded from Git
|-- README.md               Quick start and Streamlit deployment instructions
|-- DOCUMENTATION.md        This complete technical documentation
|-- .venv/                  Local virtual environment; never deployed
```

### File responsibilities

| File | Responsibility |
| --- | --- |
| `app.py` | Configures Streamlit, loads credentials, extracts PDF text, creates embeddings, searches FAISS, calls Groq, and renders the chat UI. |
| `requirements.txt` | Tells local Python and Streamlit Community Cloud which packages to install. |
| `.env.example` | Documents the names of local configuration values without containing a real secret. |
| `.env` | Stores local values such as the Groq API key. It is ignored by `.gitignore`. |
| `.gitignore` | Prevents `.env`, `.venv`, Python cache files, and compiled Python files from being uploaded to GitHub. |
| `README.md` | Provides short setup and deployment instructions. |
| `DOCUMENTATION.md` | Explains the complete architecture and source-code workflow. |

## 3. Architecture

```text
                         User browser
                              |
                              v
                     Streamlit interface
                              |
                +-------------+-------------+
                |                           |
                v                           v
          PDF file upload              User question
                |                           |
                v                           v
        pypdf text extraction       Sentence Transformer
                |                    question embedding
                v                           |
        Word-based chunking                v
                |                     FAISS search
                v                           |
        Sentence Transformer                v
        chunk embeddings              Top 3 chunks
                |                           |
                v                           v
             FAISS index --------> Retrieved context
                                                |
                                                v
                                  Groq chat completion API
                                                |
                                                v
                                      Answer in Streamlit
```

### Main technology roles

- **Streamlit**: Provides the web page, file uploader, chat input, chat
  messages, status messages, caching, and session state.
- **pypdf**: Reads text from each page of an uploaded PDF.
- **Sentence Transformers**: Converts document chunks and questions into
  numerical vectors called embeddings.
- **FAISS**: Stores document vectors and performs fast similarity search.
- **Groq Python SDK**: Sends the retrieved context and question to the
  configured Groq-hosted language model.
- **python-dotenv**: Loads local values from `.env`.
- **Python `os` module**: Reads environment variables.

## 4. Complete imports

The current source imports the following modules:

```python
import os

import streamlit as st
from pypdf import PdfReader
from sentence_transformers import SentenceTransformer
from dotenv import load_dotenv
from groq import Groq
import faiss
from streamlit.errors import StreamlitSecretNotFoundError
```

### Import-by-import explanation

#### `import os`

Uses Python's standard-library operating-system helpers. The application uses
`os.getenv()` to read local environment variables such as `GROQ_API_KEY` and
`GROQ_MODEL`.

#### `import streamlit as st`

Imports Streamlit using the conventional short name `st`. It is used for:

- Page configuration
- Titles and explanatory text
- PDF file upload
- Spinners and success/error messages
- Chat message rendering
- Chat input
- Session state
- Resource caching
- Cloud secrets
- Stopping execution when configuration or document validation fails

#### `from pypdf import PdfReader`

Imports the PDF reader class. It reads the uploaded file and exposes its pages
so that text can be extracted from each page.

#### `from sentence_transformers import SentenceTransformer`

Imports the embedding model wrapper. The application uses
`sentence-transformers/all-MiniLM-L6-v2` to create vector representations of
document chunks and user questions.

#### `from dotenv import load_dotenv`

Imports the helper that loads key-value pairs from a local `.env` file into
the process environment. This is useful for local development.

#### `from groq import Groq`

Imports the Groq API client. The client calls the Groq chat completions API
with the retrieved document context and the user's question.

#### `import faiss`

Imports Facebook AI Similarity Search. The code uses `IndexFlatIP`, an exact
inner-product index, to find the document chunks most similar to the question
embedding.

#### `from streamlit.errors import StreamlitSecretNotFoundError`

Imports the specific exception raised when Streamlit secrets are unavailable.
This allows the application to work in both environments:

- Streamlit Community Cloud, where values are stored in `st.secrets`
- Local development, where values are stored in `.env` or normal environment
  variables

## 5. Dependencies

The dependency file currently contains:

```text
streamlit
pypdf
sentence-transformers
python-dotenv
groq
faiss-cpu
```

### Why each package is required

| Package | Used for |
| --- | --- |
| `streamlit` | Web interface and application runtime |
| `pypdf` | PDF reading and text extraction |
| `sentence-transformers` | Text and question embeddings |
| `python-dotenv` | Loading local `.env` settings |
| `groq` | Groq model API requests |
| `faiss-cpu` | CPU-based vector similarity search |

`faiss-cpu` is suitable for the free Streamlit Community Cloud CPU runtime.
The first embedding-model load can take time because the model may need to be
downloaded and cached.

## 6. Configuration and secrets

The application reads two settings:

```text
GROQ_API_KEY
GROQ_MODEL
```

Example local `.env` format:

```text
GROQ_API_KEY=your_groq_api_key
GROQ_MODEL=your_supported_groq_model
```

The lookup order implemented by `get_setting()` is:

1. Try `st.secrets`, which is the correct source on Streamlit Community Cloud.
2. If Cloud secrets are unavailable, use `os.getenv()`.
3. For `GROQ_MODEL`, use `openai/gpt-oss-120b` as the default if no value is
   supplied.

The API key is required. If it is missing, the application displays an error
and calls `st.stop()` before creating a Groq client.

Never commit `.env` or a real API key to GitHub. If a key is exposed, revoke it
and create a replacement key.

## 7. Source-code workflow from start to finish

### Step 1: Configure the Streamlit page

At the top of `app.py`, `st.set_page_config()` sets:

- Browser title: `RAG Document Chatbot`
- Browser icon: a document emoji
- Layout: wide

This must happen before most other Streamlit UI calls.

### Step 2: Load configuration

`load_dotenv()` loads local `.env` values.

`get_setting(name, default=None)` then reads the requested value. It first
checks Streamlit Cloud secrets and falls back to environment variables. This
means the same source code works locally and after GitHub deployment.

The code creates:

```python
GROQ_API_KEY = get_setting("GROQ_API_KEY")
GROQ_MODEL = get_setting("GROQ_MODEL", "openai/gpt-oss-120b")
```

If no API key is found, the UI displays a setup message and stops the current
Streamlit run. If the key exists, `Groq(api_key=GROQ_API_KEY)` creates the API
client.

### Step 3: Initialize session state

Streamlit reruns the script when a user uploads a file or submits a question.
`st.session_state` preserves selected values between those reruns:

```python
st.session_state.messages
st.session_state.uploaded_file_name
```

- `messages` stores previous user and assistant messages.
- `uploaded_file_name` identifies the currently selected PDF.

When a different file is uploaded, the previous chat messages are cleared so
answers from the old document are not shown with the new document.

### Step 4: Load and cache the embedding model

`load_embedding_model()` creates:

```python
SentenceTransformer("sentence-transformers/all-MiniLM-L6-v2")
```

The `@st.cache_resource` decorator ensures that the model is reused rather
than loaded from disk or downloaded on every Streamlit rerun. This reduces
startup time and memory churn.

### Step 5: Render the initial user interface

The application renders:

```python
st.title(...)
st.write(...)
st.file_uploader(..., type=["pdf"])
```

Only PDF files are accepted by the uploader.

### Step 6: Extract text from the PDF

When a file is present, `extract_text_from_pdf(pdf_file)`:

1. Creates `PdfReader(pdf_file)`.
2. Iterates through every page.
3. Calls `page.extract_text()`.
4. Keeps non-empty page text.
5. Joins page text using newline characters.
6. Returns the complete text and page count.

If the document contains no readable text, the app displays a message saying
that the PDF may be scanned or image-based, then stops. OCR is not currently
implemented.

### Step 7: Split the document into chunks

`split_text_into_chunks(text, chunk_size=180, chunk_overlap=40)` works on
words, not characters or tokens.

For a document:

1. `text.split()` creates a list of words.
2. The step size is calculated as `180 - 40 = 140`.
3. A window of up to 180 words is collected.
4. The next window begins 140 words later.
5. Therefore, adjacent chunks overlap by approximately 40 words.

The overlap helps preserve context when an answer crosses a chunk boundary.
The function returns a list of non-empty chunk strings.

### Step 8: Create document embeddings

`create_chunk_embeddings(document_chunks, embedding_model)` calls:

```python
model.encode_document(
    chunks,
    convert_to_numpy=True,
    normalize_embeddings=True
)
```

Each chunk becomes a fixed-size numerical vector. Normalization makes inner
product search behave like cosine similarity for these vectors.

### Step 9: Build the FAISS vector index

`create_faiss_index(embeddings)`:

1. Reads the vector dimension from `embeddings.shape[1]`.
2. Creates `faiss.IndexFlatIP(embedding_dimension)`.
3. Converts the vectors to `float32`.
4. Adds all vectors to the index.

`IndexFlatIP` performs exact inner-product similarity search. The index exists
in memory for the current Streamlit script run; it is not persisted to a
database or file.

### Step 10: Display prior chat messages

Before accepting a new question, the app loops over
`st.session_state.messages` and renders each item with
`st.chat_message(message["role"])`.

This reconstructs the visible conversation after a Streamlit rerun.

### Step 11: Receive a question

`st.chat_input("Ask a question about the document")` waits for the user to
submit a question.

When submitted, the question is:

1. Added to session state as a `user` message.
2. Immediately displayed in the chat.
3. Passed to the retrieval function.

### Step 12: Embed the question and retrieve context

`retrieve_relevant_chunks()` performs the retrieval stage:

1. Encodes the question with `model.encode_query()`.
2. Converts the result to `float32`.
3. Reshapes it to a one-row matrix.
4. Calls `index.search(question_embedding, top_k)`.
5. Uses the returned indices to select the matching chunk strings.
6. Ignores `-1` indices, which indicate missing results.

The current call uses `top_k=3`, so at most three relevant chunks are sent to
the language model.

### Step 13: Build the generation prompt

`generate_answer()` joins the retrieved chunks with blank lines:

```text
chunk 1

chunk 2

chunk 3
```

It then creates:

- A system prompt that instructs the model to answer only from the document
  context.
- A user prompt containing the retrieved context and the question.

The system prompt also instructs the model to say:

```text
I could not find that information in the document.
```

when the answer is not supported by the supplied context.

### Step 14: Call the Groq model

The application calls:

```python
client.chat.completions.create(
    model=model_name,
    messages=[system_message, user_message],
    temperature=0
)
```

`temperature=0` favors consistent, less-random answers. The generated text is
read from:

```python
response.choices[0].message.content
```

### Step 15: Render and save the answer

The answer is displayed inside an assistant chat message. If the returned
answer is empty, the app reports an error and stops.

For a valid answer, the application appends:

```python
{
    "role": "assistant",
    "content": answer
}
```

to `st.session_state.messages`, so it remains visible during future reruns.

### Step 16: Handle API errors

The Groq request and answer rendering are inside a `try` block. If the API
request fails, the app shows a user-facing message asking the user to check:

- API key
- Model name
- Internet connection

It also displays the exception text using `st.caption()` to help with
debugging.

## 8. RAG data flow example

Suppose a PDF contains 1,000 words and the user asks:

```text
What is the refund policy?
```

The application does the following:

1. Extracts the 1,000 words from the PDF.
2. Produces overlapping 180-word chunks.
3. Embeds every chunk into a vector.
4. Stores those vectors in FAISS.
5. Embeds the question.
6. Searches FAISS for the three closest chunk vectors.
7. Places the three matching text chunks into the prompt.
8. Sends the prompt to the Groq model.
9. Displays the answer in the browser.

The model receives the retrieved context rather than the complete 1,000-word
document.

## 9. Streamlit rerun behavior

Streamlit executes the Python script from top to bottom on interactions such
as uploading a file or submitting chat input. The application is designed
around this behavior:

- The embedding model is protected by `st.cache_resource`.
- Conversation history is stored in `st.session_state`.
- The uploaded file is read again during the current run.
- The chunks and FAISS index are rebuilt during the current run.

This keeps the implementation simple, but it also means the app does not
persist documents or indexes between sessions.

## 10. Local execution

From the repository root:

```text
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env
streamlit run app.py
```

Edit `.env` and replace the placeholder values with a valid Groq API key and a
supported model name.

## 11. Streamlit Community Cloud execution

1. Push the repository to GitHub without `.env` and `.venv`.
2. Open [share.streamlit.io](https://share.streamlit.io/) and sign in with
   GitHub.
3. Create an app from the repository.
4. Set the main file path to `app.py`.
5. Add the following in the app's Advanced settings:

   ```toml
   GROQ_API_KEY = "your_groq_api_key"
   GROQ_MODEL = "openai/gpt-oss-120b"
   ```

6. Deploy the app.

Streamlit Community Cloud installs the packages from `requirements.txt`.
Future pushes to the selected branch trigger redeployment.

## 12. Current limitations and extension ideas

### Current limitations

- Scanned or image-only PDFs are not supported because OCR is not included.
- Uploaded files are held in the current session and are not permanently
  stored.
- The FAISS index is rebuilt for the active run instead of being persisted.
- There is no authentication or per-user document database.
- `page_count` is calculated during extraction but is not currently shown in
  the interface.
- Dependencies are not pinned to exact versions, so future package releases
  could change deployment behavior.

### Possible future improvements

- Add OCR for scanned PDFs.
- Cache chunks and indexes by uploaded-file hash.
- Display page references with retrieved chunks.
- Add support for multiple uploaded documents.
- Persist indexes in a database or object store.
- Add exact dependency versions after testing a deployment environment.
- Add a configurable chunk size, overlap, and retrieval count.
- Add automated tests for extraction, chunking, retrieval, and prompt creation.

