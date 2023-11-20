# ABAP Vector Search Library

The ABAP Vector Search Library is a Vector Database developed entirely in ABAP, designed to offer an independent solution without relying on external vector databases. It's particularly tailored for the "chat with your documents" scenario, enabling the slicing of documents into manageable chunks and the injection of these semantically relevant segments into user prompts. This approach facilitates an interactive and efficient method for handling and responding to user queries within document-based systems.

The library employs the RAG (Retrieval Augmented Generation) technique, which is integral to its functionality, enhancing the interaction between users and document data.

RAG is the technique that is powering the “chat with your documents” scenario:
 - Slice documents into chunks
 - Inject semantically relevant chunks into the Prompt:
```code
Considering only information below:
###
{chunk01}
{chunk02}
{chunk03}
###
please provide response to the following request: 
{query from user}
```

## How it works? Why it works? O_O
Explained here: [implementation](implementation.md).

## Sample Usage

### Query DB:
<details>
<summary>show code...</summary>

```ABAP
    DATA(lo_lib) = zcl_vdb_002_lib=>new( ). "instantiate library

    DATA(ls_v) = lo_lib->read_vector( id ). "read vector from DB by its GUID
    DATA(lt_q) = lo_lib->query( ls_v-q1b ). "query DB with the quantized 1536-dimensional vector

    cl_demo_output=>display( lt_q ). "observe ranked result
```
</details>

### Answer with RAG:
<details>
<summary>show code...</summary>

```abap
    DATA(lo_e) = zcl_vdb_002_embedding_full=>new( ). "using https://github.com/microsoft/aisdkforsapabap

    DATA(lv_text) = 'Answer to the Ultimate Question of Life, the Universe, and Everything'.

* Embed and save text, retrieve ID
    DATA(lv_id_q) = lo_e->embed_and_save( lv_text )-id.
    COMMIT WORK AND WAIT.

* Generate an answer as vector
    DATA(lx_a) = lo_e->answer( lv_text ).
*--------------------------------------------------------------------*
    DATA(lo_lib) = zcl_vdb_002_lib=>new( ).
    DATA(lt_q) = lo_lib->query_by_id( lv_id_q ). "get chunks semantically close to the question
* Query for answers
    DATA(lt_a) = lo_lib->query( lx_a ).          "get chunks semantically close to the "modelled answer"

* Combine, sort, and remove duplicates from query results
    DATA(lt_all) = lt_q.
    APPEND LINES OF lt_a TO lt_all.
    SORT lt_all BY id.
    DELETE ADJACENT DUPLICATES FROM lt_all.

* Prepare for RAG (Retrieve and Generate) method
    DATA: lv_prompt TYPE string.
    DATA: lt_used TYPE zcl_vdb_002_lib=>tt_vector.

* Execute RAG with text and combined results, get prompt and used vectors
    DATA(lv_rag_answer) = lo_e->rag(
                            EXPORTING iv_ = lv_text
                                      it_ = lt_all
                            IMPORTING ev_prompt = lv_prompt
                                      et_used   = lt_used
    ).

    cl_demo_output=>write( lv_rag_answer ). "observe RAG answer
    cl_demo_output=>write( lv_prompt ).     "check out the resulting prompt
    cl_demo_output=>display(  ).
```

</details>

## Dependencies

The library is self-contained for the core functionality of the Vector Search.
However if you want to generate embeddings, you might want to install this [OpenAI ABAP SDK](https://github.com/microsoft/aisdkforsapabap).

To instantiate SDK you might need to use your own private keys, you can store them locally in a simple key-value .env file, something like:

```.env
#https://<subdomain>.openai.azure.com/openai/deployments/<api_dep>/chat/completions?api-version=2023-07-01-preview
#https://<subdomain>.openai.azure.com/openai/deployments/<api_dep>/chat/completions?api-version=<API_VER>
API_URL=https://<subdomain>.openai.azure.com/
API_VER=2023-07-01-preview
API_KEY=<xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx>
API_DEP=<deployment_id>
API_DEP_EMBED=<deployment_id_embedding_model>
```


```ABAP
    DATA(lo_) = zcl_vdb_000_dotenv=>new( lv_path ).
    DATA(lv_api_key) = cut->v( 'API_KEY' ).

    data(ls_) = VALUE ts_env(
      api_url       = lo_->v( k = 'API_URL' )
      api_ver       = lo_->v( k = 'API_VER' )
      api_key       = lo_->v( k = 'API_KEY' )
      api_dep       = lo_->v( k = 'API_DEP' )
      api_dep_embed = lo_->v( k = 'API_DEP_EMBED' )
    ).

```
## Installation

Install this repository using [abapGit](https://github.com/abapGit/abapGit#abapgit).

© 2023 Alice Vinogradova. For more information, see the [LICENSE](LICENSE) file in this repository.
