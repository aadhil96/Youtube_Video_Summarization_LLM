# LangChain: Summarize Text From YouTube or Website

This Streamlit application allows you to summarize the content from a YouTube video or a website using the LangChain library and the Groq API.

## Screenshot
![screenshot](https://github.com/aadhil96/Youtube_Video_Summarization_LLM/blob/3c7afd9429f6f1c948defdab717ce6a176df75b9/screenshot.JPG)

## Features

- Summarize text from YouTube videos.
- Summarize text from websites.
- Integration with the Groq API for language model processing.
- User-friendly interface with Streamlit.

## Prerequisites

Before running the application, ensure you have the following prerequisites:

- Python 3.7 or higher.
- Streamlit installed.
- LangChain library installed.
- Validators library installed.
- Groq API key.

## Installation

1. Clone the repository:

    ```bash
    git clone https://github.com/your-username/your-repo.git
    cd your-repo
    ```

2. Install the required Python packages:

    ```bash
    pip install streamlit langchain validators langchain-groq langchain-community
    ```

## Usage

1. Run the Streamlit application:

    ```bash
    streamlit run your_script_name.py
    ```

2. Open the provided URL in your web browser.

3. Enter your Groq API key in the sidebar.

4. Enter the URL of the YouTube video or website you want to summarize.

5. Click the "Summarize the Content from YT or Website" button.

6. Wait for the application to process the content and display the summary.

## Code Overview

The main components of the code are as follows:

- **Streamlit Configuration**: Sets up the Streamlit page configuration and title.
- **Input Fields**: Collects the Groq API key and the URL to be summarized.
- **Groq Model Initialization**: Initializes the language model using the Groq API.
- **Prompt Template**: Defines the prompt template for summarization.
- **Summarization Logic**: Validates inputs, loads the content from the URL, and summarizes it using the LangChain library.

### Example Code Snippet

```python
import validators, streamlit as st
from langchain.prompts import PromptTemplate
from langchain_groq import ChatGroq
from langchain.chains.summarize import load_summarize_chain
from langchain_community.document_loaders import YoutubeLoader, UnstructuredURLLoader

## Streamlit APP
st.set_page_config(page_title="LangChain: Summarize Text From YT or Website", page_icon="🦜")
st.title("🦜 LangChain: Summarize Text From YT or Website")
st.subheader('Summarize URL')

## Get the Groq API Key and URL (YT or website) to be summarized
with st.sidebar:
    groq_api_key = st.text_input("Groq API Key", value="", type="password")

generic_url = st.text_input("URL", label_visibility="collapsed")

## Gemma Model Using Groq API
llm = ChatGroq(model="Gemma-7b-It", groq_api_key=groq_api_key)

prompt_template = """
Provide a summary of the following content in 300 words:
Content:{text}
"""
prompt = PromptTemplate(template=prompt_template, input_variables=["text"])

if st.button("Summarize the Content from YT or Website"):
    ## Validate all the inputs
    if not groq_api_key.strip() or not generic_url.strip():
        st.error("Please provide the information to get started")
    elif not validators.url(generic_url):
        st.error("Please enter a valid URL. It can be a YouTube video URL or website URL")
    else:
        try:
            with st.spinner("Waiting..."):
                ## Loading the website or YouTube video data
                if "youtube.com" in generic_url:
                    loader = YoutubeLoader.from_youtube_url(generic_url, add_video_info=True)
                else:
                    loader = UnstructuredURLLoader(urls=[generic_url], ssl_verify=False,
                                                   headers={"User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 13_5_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/116.0.0.0 Safari/537.36"})
                docs = loader.load()

                ## Chain For Summarization
                chain = load_summarize_chain(llm, chain_type="stuff", prompt=prompt)
                output_summary = chain.run(docs)

                st.success(output_summary)
        except Exception as e:
            st.exception(f"Exception:{e}")
