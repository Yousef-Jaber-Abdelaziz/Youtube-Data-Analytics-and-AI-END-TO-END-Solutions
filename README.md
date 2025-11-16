# YouTube Trending Video Analytics – End-to-End BI Pipeline with an Intelligent RAG Agent

## 📌 Project Description
This project is an end-to-end, on-premise data solution built in two distinct phases. It starts with a robust data engineering pipeline using Microsoft technologies to analyze the **YouTube Trending Video Dataset** (focusing on Canada, the US, and Brazil) and culminates in a sophisticated, conversational AI agent for data exploration.

* **Phase 1: The Data Warehouse Foundation**
    An on-premise Data Engineering pipeline was built using **SSIS, SQL Server, and SSAS**. This pipeline ingests raw CSV files, transforms them through ODS and STG layers, and builds a **star-schema data warehouse (DWH)**. The data is then served to an interactive **Power BI dashboard** via an SSAS semantic model.

* **Phase 2: The Intelligent RAG Agent**
    To make the data and project metadata more accessible, an intelligent agent was built using **Python, LangChain, and Groq (LLM)**. This agent provides a single, natural-language interface (built with Streamlit) to query the entire data ecosystem. It can:
    1.  **Answer conceptual questions** about the project (from documentation).
    2.  **Generate and run T-SQL** queries against the Data Warehouse.
    3.  **Generate and run DAX** queries against the SSAS Semantic Model.

This project was completed as part of my training in **Global Brand Solutions Academy**.

---

## 💻 Technology Stack

| Area | Phase 1 (Data Engineering) | Phase 2 (AI Agent) |
| :--- | :--- | :--- |
| **ETL** | SQL Server Integration Services (SSIS) | Python (Pandas for preprocessing) |
| **Database** | MS SQL Server (for ODS, STG, DWH) | SQLAlchemy (for SQL connection) |
| **Data Model** | SSAS Tabular (Semantic Layer) | ADODBAPI (for DAX connection) |
| **Visualization** | Power BI | Streamlit (for Agent UI) |
| **AI/LLM** | - | LangChain (Orchestration) |
| **LLM** | - | Groq (Reasoning & Generation) |
| **Vector DB** | - | FAISS (for RAG) |
| **Embeddings** | - | HuggingFace (`all-MiniLM-L6-v2`) |

---

## 🏗️ Phase 1: The Data Warehouse Foundation

This phase established the core BI architecture, moving data from raw files to an interactive dashboard.

### Project Overview (Phase 1)
1.  **Getting the Data (SSIS)** → Extracted CSV files for (Canada, US, and Brazil).
2.  **ODS Layer (SSIS)** → Loaded raw data into the ODS as the initial layer.
3.  **STG Layer (SSIS)** → Performed cleansing, data conversion, and standardization.
4.  **DWH Layer (SSIS)** → Built star schema, added surrogate keys, and created lookups between Fact & Dimensions.
5.  **Semantic Model (SSAS)** → Created relationships and DAX measures for KPIs (views, likes, dislikes, comments).
6.  **Dashboard (Power BI)** → Designed interactive visuals to analyze trends.

---

### 🗂 Data Extraction Challenges

While extracting the dataset into **SSIS**, we faced an issue with the **Connection Manager**. Some columns were incorrectly detected as **object** instead of their actual data type.

🔎 After analyzing the problem, we found that the **`title`** column contained text values with embedded commas. These commas confused the CSV parser, causing SSIS to misinterpret column structures.

✅ **Solution:**
We used **Python preprocessing** to fix the issue before loading into SSIS. Specifically, we added quotes (`""`) around any text field containing at least one comma. This ensured that SSIS correctly treated the values as a single column.

👉 Check the detailed implementation here: [![Open Notebook](https://img.shields.io/badge/Jupyter-Notebook-orange?logo=jupyter)](fixing-the-youtube-title-feature.ipynb)

---

### 🔹 Step 1: Getting the Data & Creating the ODS Layer

After fixing the **title column issue**, we successfully created the **Connection Manager** for the CSV files. We created a new database called **`YoutubeODS`** and loaded the **unioned** data from Canada, US, and Brazil into the **`YoutubeODS.Videos`** table.

<p align="center">
  <img src="Full Flow ScreenShots/1) ETL Pipeline/ODS.png" alt="ODS Data Flow" width="500"/>
</p>

---

### 🔹 Step 2: Creating the STG Layer

Next, we designed the **STG database structure** to prepare the data for the Data Warehouse. We created both a **Logical Model** and a **Conceptual Model** to define the dimensions and fact table. Based on these, we created the **`YoutubeSTG`** database and used SSIS to populate the dimension and fact tables.

<p align="center">
  <img src="Data Models/Logical Model.png" alt="Logical Model" width="400" height="400"/>
  <img src="Data Models/Conceptual model.png" alt="Conceptual Model" width="400"/>
</p>
<p align="center">
  <img src="Full Flow ScreenShots/1) ETL Pipeline/STG Full View.png" alt="STG Data Flow" width="500"/>
</p>

---

### 🔹 Step 3: Creating the DWH Layer

After finalizing the STG layer, we built the **Data Warehouse (DWH)**. We created a new database called **`YoutubeDWH`** and transferred the data from STG. Using SSIS, we performed **lookups** between the Fact and Dimensions so the Fact table could reference the correct **surrogate keys** (IDs).

<p align="center">
  <img src="Full Flow ScreenShots/1) ETL Pipeline/STG Full View.png" alt="Full DWH View" width="400"/>
  <img src="Full Flow ScreenShots/1) ETL Pipeline/DWH Fact Lookup.png" alt="DWH Lookup" width="400"/>
</p>

---

### 🔹 Step 4: Building the Semantic Model (SSAS)

With the DWH in place, we loaded the **Fact** and **Dimension** tables into an **SSAS Tabular Project** to build the **Semantic Model**. This involved creating **DAX measures** (total views, likes, etc.) and defining **relationships** before deploying the model to the SSAS instance.

<p align="center">
  <img src="Full Flow ScreenShdcoots/2) Semantic Model (SSAS)/Semantic Layer Model View.png" alt="SSAS Semantic Model View" width="500"/>
</p>

---

### 🔹 Step 5: Power BI Interactive Dashboard

Finally, we connected **Power BI** to the deployed **SSAS Semantic Model**. The resulting two-page dashboard provides insights into video performance, regional comparisons, and trending patterns.

<p align="center">
  <img src="Full Flow ScreenShots/3) Dashboard Screenshots/Data Overview.png" alt="Power BI Dashboard - Page 1" width="400"/>
  <img src="Full Flow ScreenShots/3) Dashboard Screenshots/Channels Analysis.png" alt="Power BI Dashboard - Page 2" width="400"/>
</p>

---

## 🧠 Phase 2: The Intelligent RAG Agent

This phase brings the project to life with a conversational interface. The agent acts as a single "front door" to all project assets: the documentation, the DWH, and the SSAS model.



### How the Agent Works

When a user submits a query (e.g., in a Streamlit app), a multi-step process begins:

#### 1. Query Normalization
The agent first passes the query to an LLM to "normalize" it, ensuring database compatibility.
* **User:** "What are the total views for the US and UK?"
* **Normalized:** "What are the total views for the **United States of America** and **United Kingdom**?"

#### 2. Intent Routing
The agent uses an LLM to classify the user's intent into one of three categories:
* **`RAG`**: For conceptual questions ("How was the DWH built?", "What was the CSV challenge?").
* **`SQL`**: For raw data requests ("List the top 10 channels by view count").
* **`DAX`**: For metric/KPI questions ("What is the total engagement rate?", "Show me views for Brazil").

#### 3. Execution via Specialized Chains

Based on the intent, the agent routes the query to a specific chain:

* **If Intent = `RAG` (Documentation Query)**
    1.  The agent retrieves relevant chunks from a **FAISS vector database** (which was pre-loaded with the project's PDF documentation).
    2.  The chunks and the question are passed to the LLM.
    3.  The LLM generates a human-readable answer based *only* on the provided context.

* **If Intent = `SQL` (Data Warehouse Query)**
    1.  **Dynamic Schema Retrieval:** The agent *first* uses RAG to query its FAISS DB for information about the DWH schema (table names, columns, relationships).
    2.  **SQL Generation:** It feeds the user's question and the retrieved schema to the LLM, instructing it to generate optimized, safe T-SQL (including overflow protection like `CAST(SUM(Views) AS BIGINT)`).
    3.  **Execution:** The generated T-SQL is executed against the `YoutubeDWH` database via SQLAlchemy.
    4.  The raw `pd.DataFrame` result is returned.

* **If Intent = `DAX` (Semantic Model Query)**
    1.  **Find Existing Measure:** The agent *first* searches the FAISS DB to see if a pre-defined DAX measure (like `[Total Views]`) can answer the question.
    2.  **Generate New Measure:** If no existing measure is found, the agent (using its RAG-retrieved knowledge of the SSAS model) generates a new DAX query (e.g., `EVALUATE ROW("Result", CALCULATE(SUM(VideosFact[ViewCounts]), Country[Country] = "Brazil"))`).
    3.  **Execution:** The DAX query is executed against the `YoutubeAnalysis` SSAS model via `adodbapi`.
    4.  The raw `pd.DataFrame` result is returned.

#### 4. Polished Response
The raw data (DataFrame or RAG text) from the execution step is sent to a final LLM chain. This chain formats the data into a friendly, conversational response, complete with Markdown tables, clear explanations, and a suggested follow-up question.
