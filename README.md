# RAG Document Chatbot

A Streamlit document question-answering app using PDF text extraction,
Sentence Transformers embeddings, FAISS similarity search, and Groq.

For the full repository map, architecture, imports, dependencies, and
end-to-end source-code workflow, read
[DOCUMENTATION.md](DOCUMENTATION.md).

## Run locally

1. Create and activate a Python virtual environment.
2. Install the dependencies:

   ```text
   pip install -r requirements.txt
   ```

3. Copy `.env.example` to `.env` and add your Groq API key.
4. Start the app:

   ```text
   streamlit run app.py
   ```

The `.env` file is ignored by Git and must never be committed.

## Deploy free on Streamlit Community Cloud

Streamlit Community Cloud can deploy this app directly from a public or
private GitHub repository. The hosting is free within Streamlit's current
usage limits; the Groq API is a separate service and is subject to Groq's
availability and limits.

1. Push this project to GitHub. Do not push `.env` or `.venv`.
2. Open [share.streamlit.io](https://share.streamlit.io/) and sign in with
   GitHub.
3. Select **Create app** and choose **Deploy a public app from GitHub**.
4. Select your repository and branch, then set **Main file path** to
   `app.py`.
5. Open **Advanced settings**, choose the Python version used by your
   repository, and add these secrets in TOML format:

   ```toml
   GROQ_API_KEY = "your_groq_api_key"
   GROQ_MODEL = "openai/gpt-oss-120b"
   ```

6. Select **Deploy**. Streamlit installs the packages from
   `requirements.txt` and gives you a shareable `streamlit.app` URL.

To update the deployed app, push changes to the selected GitHub branch.
Streamlit Community Cloud automatically redeploys the app.

## Important limitations

- Uploaded PDFs and chat history are kept in the browser session only; they
  are not a permanent database.
- The app requires an internet connection for the Groq API and for the
  initial Sentence Transformers model download.
- Never expose or commit API keys. If a key is accidentally exposed, revoke
  it in Groq and create a new one.
