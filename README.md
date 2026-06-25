# Introduction - Azure SQL Hyperscale for AI and RAG Workloads 

This workshop is built on **Azure SQL Hyperscale**, a cloud-native database architecture designed for high-performance, scalable, and AI-driven applications. 


## Why Hyperscale? 

Azure SQL Hyperscale separates compute and storage, enabling: 

- Independent scaling of compute and storage 
- Rapid database growth (up to 100 TB) 
- High-performance workloads with minimal latency 
- Fast database backups and restores 

## Key features for AI scenarios 

This workshop leverages Hyperscale capabilities to support modern AI patterns: 

- **Vector support** for semantic search and embeddings 
- **High throughput** for retrieval workloads (RAG) 
- **Separation of compute and storage** for scalable AI pipelines 
- **Integration with Azure services** such as: 
  - Azure OpenAI 
  - Microsoft Fabric 
  - Microsoft Foundry Agents 

## Why it matters for this lab 

In this workshop, Azure SQL Hyperscale enables:
 
- Efficient storage of **FAQ content + vector embeddings** 
- Fast execution of **semantic search queries** 
- Scalable backend for **Retrieval-Augmented Generation (RAG)** 

You'll use Hyperscale as the foundation for: 
- Vector search 
- AI grounding 
- Agent orchestration 
- Analytics with Fabric 

> In real-world AI applications, Hyperscale is ideal when combining transactional + AI + analytics workloads. 

=== 

# Exercise 1 - AI-enhanced querying with Azure SQL Hyperscale 

## Learning objective 

In this exercise, you'll use **Visual Studio Code** to connect to **Azure SQL Hyperscale**, inspect the FAQ knowledge base, and run a **semantic search query using vector embeddings**. 

By the end of this exercise, you'll be able to: 

- Connect to Azure SQL using VS Code. 
- Explore FAQ data stored in the database. 
- Inspect vector embeddings. 
- Run a semantic similarity query to retrieve relevant FAQ answers. 

--- 

# Steps 

1. Open **Visual Studio Code** from the desktop. 

1. Select the **SQL Server** icon in the Activity Bar. 

   ![itosldk0.png](../../media/itosldk0.png) 

1. In **Connections**, select **+ Add Connection**. 

   ![hqyq7nso.png](../../media/hqyq7nso.png) 

1. Configure the connection to the workshop database. 

  - In the **Connect to Database** dialog, use the following values: 

  | Setting                 | Value                                                       | 
  | ----------------------- | ----------------------------------------------------------- | 
  | **Profile Name**        | `FAQ`                                                       | 
  | **Input type**          | `Parameters`                                                | 
  | **Server name**         | `faq-ai-assistant-@lab.LabInstance.Id.database.windows.net` | 
  | **Authentication type** | `Microsoft Entra ID - Universal with MFA support`           | 
  | **Database name**       | `faq-ai-assistant-db`                                       | 
  | **Encrypt**             | `Mandatory`                                                 |  

    > This lab uses **Azure SQL Hyperscale** for scalable AI workloads. 

  ![642dyvix.png](../../media/642dyvix.png) 

1. Select **Sign in** and authenticate with your Microsoft Entra ID account: 

   **Username**: `@lab.CloudPortalCredential(User1).Username` 
   **TAP**: `@lab.CloudPortalCredential(User1).AccessToken` 

	![6ggh01ry.png](../../media/6ggh01ry.png) 

	> Close the browser tab after signing in. 

1. Select **Connect**. In the SQL Server extension panel, hover over the new connection and verify that the **faq-ai-assistant-db** database is available. 

   ![jye888g9.png](../../media/jye888g9.png) 

   ![y4cal7ce.png](../../media/y4cal7ce.png) 

1. After the connection is created, open a **new SQL query window**. 

   - Select **Ctrl + Shift + P** 
   - Select: `MS SQL: New Query` 

1. Ensure the query window is connected to the database **faq-ai-assistant-db**. You should see the database name displayed in the **bottom status bar**. 

   ![i2ys8njv.png](../../media/i2ys8njv.png) 

--- 

# Explore the FAQ knowledge base 

1. In the query window, run the following query to view the FAQ content table. 

	
sql
    SELECT TOP 10 *
    FROM dbo.FAQ_Content;
    ```

  ![2hl8shze.png](../../media/2hl8shze.png) 

1. Review the results returned. 

    - You should see columns such as **faq_id**, **category**, **question**, and **answer**. 
    - Example: 

    | faq_id | category | question                 | 
    | ------ | -------- | ------------------------ | 
    | 1      | Orders   | How do I track my order? | 

	> This table stores the **FAQ knowledge base used by the AI assistant**. 

  ![x97lg4nx.png](../../media/x97lg4nx.png) 

--- 

# Inspect the embeddings table 

1. Run the following query to view the embeddings table. 

	
sql
    SELECT TOP 5 *
    FROM dbo.FAQ_Embeddings;
    
 

  ![aidtgme1.png](../../media/aidtgme1.png) 

1. Review the results returned. 

	- You should see: 

    | faq_id | question_embedding | 
    | ------ | ------------------ | 

![fulfasu6.png](../../media/fulfasu6.png) 

>[!note]The column **question_embedding** stores a **vector representation of each question**. 

>[!note]These vectors were generated earlier using **Azure OpenAI embeddings**. 

>[!note]Each embedding contains **1536 numeric values representing semantic meaning**. 

--- 

# Validate data loaded correctly 

1. Run the following query to confirm how many FAQ records exist. 

	
sql
	SELECT COUNT(*) AS faq_count
    FROM dbo.FAQ_Content;
    ```

    ![8xyajaja.png](../../media/8xyajaja.png) 

1. Run the following query to confirm how many embeddings were loaded. 

	
sql
    SELECT COUNT(*) AS embedding_count
    FROM dbo.FAQ_Embeddings;
    ```

  ![ulfxwm8h.png](../../media/ulfxwm8h.png) 

1. Compare the results. 

	- Both counts should match. 
	- This confirms every FAQ question has a corresponding embedding. 

--- 

# Run a semantic search query 

1. Assume a user states the following: 

	
-nocode
    My product arrived damaged.
    ```

1. To simulate semantic search, call the stored procedure which generates the query embedding and performs the vector search for you. 

	
sql
    EXEC dbo.SearchFAQ @user_question = N'My product arrived damaged';
    ```

 ![61ohnvfq.png](../../media/61ohnvfq.png) 

Review the results returned. The procedure returns the most relevant FAQ rows (by default the top 3) and typically includes these columns: 

- **faq_id**: the FAQ record identifier 
- **category**: FAQ category (Orders, Returns, and so on) 
- **question**: the stored FAQ question 
- **answer**: the stored answer text 

Results are ordered by semantic similarity (cosine distance) so the top rows are the most relevant. You should see entries such as: 

- **How do I return a damaged item?** 
- **What if I received the wrong item?** 

Notice that the wording does not need to match exactly. Vector search finds similar meaning rather than exact keywords. 

The **@user_question** input is sent to the embedding service inside the procedure and the resulting vector is used to search **dbo.FAQ_Embeddings**. 

--- 

# Try another example 

1. Imagine a user asks the following question: 

	
-nocode
    Where can I check my delivery status?
    ```

1. Call the stored procedure with the new question: 

	
sql
    EXEC dbo.SearchFAQ @user_question = N'Where can I check my delivery status?';
    
 

    ![825sn0j4.png](../../media/825sn0j4.png) 

1. Review the results. You should see an FAQ similar to: 

	
-nocode
    How do I track my order?
    
 

--- 

# Compare: Traditional keyword search vs semantic (embedding) search 

1. Imagine a user asks the following question: 

    
-nocode
    Where can I check my delivery status?
    ```

1. Use the traditional keyword search (may miss relevant rows): 

	
sql
    SELECT TOP 3 c.faq_id, c.category, c.question, c.answer
    FROM dbo.FAQ_Content "c" 
    WHERE c.question LIKE N'%delivery status%';
    ```

    ![nzy9x1xu.png](../../media/nzy9x1xu.png) 

    - Example: 

	
-nocode
    -- No rows returned (keyword mismatch) 
    ```

1. Now use the semantic (embedding) search: 

	
sql
    EXEC dbo.SearchFAQ @user_question = N'Where can I check my delivery status?';
    ```

    ![75aj45t4.png](../../media/75aj45t4.png) 

    - Example (illustrative expected top result): 

	
-nocode
    faq_id | category | question                 | answer 
    -------+----------+--------------------------+---------------------------------------------- 
    1      | Orders   | How do I track my order? | You can track your order from the Orders page... 
    ```

	>[!note] The stored proc understands intent and returns the relevant FAQ. 

You've completed this exercise. Select **Next** to continue. 

=== 

# Exercise 2 - Accelerate SQL development with GitHub Copilot 

## Learning objective 

In this exercise, you'll use **GitHub Copilot and Copilot Chat inside Visual Studio Code** to accelerate SQL development for the FAQ assistant. 

By the end of this exercise, you'll be able to: 

- Leverage GitHub **Copilot Chat** in VS Code. 
- Ask Copilot to generate a semantic search query. 
- Ask Copilot to explain and improve SQL. 
- Refine AI-generated SQL before using it in the workshop. 

--- 

## Steps 

1. Open **Edge** and go to: `https://github.com/enterprises/skillable-events/sso` 

1. Select **Continue**, then sign in with your Microsoft Entra Id account: 

	**Username**: `@lab.CloudPortalCredential(User1).Username` 
    **TAP**: `@lab.CloudPortalCredential(User1).AccessToken` 

	>[!alert] If you are returned to the sign in page, please wait a few minutes and then try again. It can take some time for the account to be enabled in GitHub.

1. Return to **VS Code** and select **Signed out** at the bottom. 

    - Select **Sign in to use AI Features**. 

  ![m4g65ghk.png](../../media/m4g65ghk.png) 

1. Select **Continue with GitHub**, then select **Continue** on the **Authorize Visual Studio Code** page. 

    ![o2w38f6t.png](../../media/o2w38f6t.png) 

1. Select **Authorize Visual-Studio-Code**, then select **Open**. 

1. In VS Code, open a new SQL query window. 

   - Select **Ctrl + Shift + P** 
   - Select: **MS SQL: New Query** 

1. Open Copilot chat in VS Code. 

    - If the chat window isn't showing on the right, select **View**, then select **Chat**. 

   ![6e168u1u.png](../../media/6e168u1u.png) 

1. In Copilot Chat, enter the following prompt: 

	
chat
    Generate a T-SQL query for Azure SQL that returns the top 3 FAQ items most relevant to a user question by joining dbo.FAQ_Content and dbo.FAQ_Embeddings and ordering by VECTOR_DISTANCE using cosine similarity.
    ```

	>[!note]If you see this notification, select **Allow in this Session**. 
        ![bmrnidh5.png](../../media/bmrnidh5.png)

1. Review the SQL returned by Copilot. 

   - Do not run it yet. 
   - Check whether it includes: 

     - **dbo.FAQ_Content** 
     - **dbo.FAQ_Embeddings** 
     - A join on **faq_id** 
     - **TOP 3** 
     - **VECTOR_DISTANCE** 

1. Copy the Copilot-generated SQL into your query window or SQL file. 

1. Compare the Copilot-generated SQL with the semantic search query from **Exercise 1**. 

   - Look for similarities and differences. 
   - Notice whether Copilot used the correct `VECTOR_DISTANCE` argument order. 

   Azure SQL expects the metric as the first argument to `VECTOR_DISTANCE`, followed by the two vector values. If needed, keep the workshop's validated query as the final version. 

1. Ask Copilot to explain the query in simpler language. 

   - In Copilot Chat, enter: 

	
chat
    Explain this SQL query step by step, to someone who is new to vector search in Azure SQL.
    ```

1. Review the explanation returned by Copilot. 

    - Notice how Copilot breaks down: 

      - The join between the two tables 
      - The similarity calculation 
      - Why the results are ordered by vector distance 

1. Now ask Copilot to improve the readability of the query. 

    - Enter: 

	
chat
    Rewrite this query to make it easier to read for a workshop demo. Add clean formatting and short inline comments.
    ```

1. Copy the improved version into your SQL file. 

1. Ask Copilot for schema improvement suggestions. 

    - Enter: 

	
chat
    Review schema with dbo.FAQ_Content and dbo.FAQ_Embeddings and suggest improvements for maintainability, performance, and readability in Azure SQL.
    
 

1. Review the suggestions returned. 

    - Look for ideas such as: 

      - Readability and docs (adding helpful comments) 
      - Indexing considerations 
      - Separation of content and embeddings 
      - Column types 

1. Ask Copilot to create a reusable stored procedure draft. 

    - Enter: 

	
chat
    Generate a stored procedure draft for Azure SQL called dbo.usp_GetTopFaqMatches that accepts a query vector and returns the top 3 matching FAQs from dbo.FAQ_Content and dbo.FAQ_Embeddings. 
    ```

1. Review the stored procedure returned by Copilot. 

    - You don't need to deploy it yet. 
    - This step is meant to show how Copilot can accelerate SQL authoring for repeatable patterns. 

1. In Copilot Chat, ask one final prompt: 

	
chat
    Summarize in 3 bullet points how GitHub Copilot helped improve SQL development in this exercise.
    ```

1. Review the summary. 

    - Notice that Copilot can help with: 

      - generating SQL 
      - explaining SQL 
      - refining SQL structure 

You've completed this exercise. Select **Next** to continue. 

=== 

# Exercise 3 - Implement Retrieval-Augmented Generation (RAG) with Azure SQL Hyperscale 

## Learning objective 

In this exercise, you'll implement a simple **Retrieval-Augmented Generation (RAG)** workflow using **Azure SQL Hyperscale**. 

You'll: 

- Pass a **natural language question**. 
- Retrieve the most relevant FAQ entries from Azure SQL Hyperscale. 
- Assemble the retrieved answers into a **grounding context**. 
- Build a **grounded prompt** inside SQL. 
- Send the prompt to **GPT-4o**. 
- Review the AI-generated answer. 

By the end of this exercise, you'll understand how Azure SQL Hyperscale supports the **retrieval and prompt orchestration steps** of a RAG workflow. 

--- 

## Scenario 

Contoso Support wants to build an FAQ assistant that answers customer questions using only approved support content. 

To accomplish this, the system must: 

1. Accept a user question in natural language.   
2. Retrieve relevant FAQ records.   
3. Combine those records into a grounding context.   
4. Build a prompt for an AI model.   
5. Generate a grounded response using GPT-4o. 

--- 

# Part 1 - Retrieve FAQ data and build the grounding context 

## Steps 

1. In **VS Code**, open a new SQL query window. 

   - Select **Ctrl + Shift + P** 
   - Select: **MS SQL: New Query** 

1. Execute the following stored procedure that retrieves relevant FAQ matches for the user question:   

	
sql
    EXEC dbo.SearchFAQ @user_question = N'My product arrived damaged';
    ```

   ![asgbm8qw.png](../../media/asgbm8qw.png) 

1. Review the results returned. 

   - You should see the top matching FAQ records. 
   - Expected examples may include: 

     - How do I return a damaged item? 
     - What if I received the wrong item? 

1. Now, build a SQL script that stores the same user question in a variable and prepares a context block from the top matching FAQ records. 

    - Add the following script to a new SQL query window: 

    >[!alert] To ensure the script is copied correctly, do not use the **Type** option below. Use **Copy**, then paste the script into the query window. 

	
sql
    DECLARE @user_question NVARCHAR(1000) = N'My product arrived damaged';
    DECLARE @context NVARCHAR(MAX);
    DECLARE @prompt NVARCHAR(MAX);

    CREATE TABLE #searchResults (
      faq_id INT,
      category NVARCHAR(200),
      question NVARCHAR(MAX),
      answer NVARCHAR(MAX)
    );

    INSERT INTO #searchResults (faq_id, category, question, answer)
    EXEC dbo.SearchFAQ @user_question = @user_question;

    SELECT @context =
    (
      SELECT STRING_AGG(
          CONCAT(
            'Question: ', question, CHAR(10),
            'Answer: ', answer
          ),
          CHAR(10) + CHAR(10)
      )
      FROM #searchResults
    );

    SET @prompt =
    N'Use ONLY the context below to answer the question.

    Context:
    ' + ISNULL(@context, N'No relevant FAQ context found.') + N'

    Question:
    ' + @user_question + N'

    If the answer is not in the context, say you do not know.';

    -- SELECT @context AS retrieved_context,
    --       @prompt AS grounded_prompt;

    SELECT  @prompt AS grounded_prompt;

    DROP TABLE #searchResults;
    ```

    ![dafil5q9.png](../../media/dafil5q9.png) 

1. Run the script. 

  ![ybw540og.png](../../media/ybw540og.png) 

1. Review the **grounded_prompt** result. 

	> Select the ![fzu2qqlz.png](../../media/fzu2qqlz.png) **Switch to Text View** button. 

	- This prompt contains:  
        - The retrieved FAQ context
        - The user question
        - Instructions that force the AI to stay grounded in the data 


 ![15j1zx0b.png](../../media/15j1zx0b.png) 

--- 

# Part 2 - Send the prompt to GPT-4o 

Now that the **grounded prompt** has been created, you'll send it to **GPT-4o** from Azure SQL Hyperscale. 

1. Append the following query in the existing query editor window: 

	>[!alert] To ensure the script is copied correctly, do not use the **Type** option below. Use **Copy**, then paste the script into the query window. 

	
sql
    -- Build minimal chat-style payload
    DECLARE @payload NVARCHAR(MAX);
    DECLARE @response NVARCHAR(MAX);
    DECLARE @headers NVARCHAR(MAX) = N'{"api-key": "@lab.Variable(apikey)"}';

    SET @payload = N'{' +
      N'"messages":[' +
          N'{"role":"system","content":"You are a helpful assistant that answers using ONLY the provided context."},' +
          N'{"role":"user","content":' + N'"' + STRING_ESCAPE(@prompt, 'json') + N'"' + N'}' +
      N'],' +
      N'"temperature":0' +
    N'}';

    -- Call the chat/completions endpoint (example)
    EXEC sp_invoke_external_rest_endpoint
      @method = 'POST',
      @url = N'@lab.Variable(chaturl)',
      @headers = @headers,
      -- For production, prefer: @credential = N'your_db_scoped_credential',
      @payload = @payload,
      @response = @response OUTPUT;

    -- Show raw response and attempt common extractions
    SELECT @response AS raw_response,
          COALESCE(
            JSON_VALUE(@response, '$.result.choices[0].message.content'),
            JSON_VALUE(@response, '$.choices[0].message.content'),
            JSON_VALUE(@response, '$.output[0].content[0].text'),
            @response
          ) AS ai_answer;

    -- If you want to extract structured JSON returned by the model (see sample repo),
    -- you can use OPENJSON to parse `$.result.choices[0].message` or similar paths.
    ```

  ![zs6vmxo7.png](../../media/zs6vmxo7.png)

1. Run the script and view the response.

  ![bz3fdrus.png](../../media/bz3fdrus.png)

  <!-- Notes: 

  - Prefer using a database-scoped credential via the **@credential** parameter for **sp_invoke_external_rest_endpoint** instead of placing API keys in the script. --> 

### Test additional questions 

1. Modify the user question. 

    Example: 

	
sql
    DECLARE @user_question NVARCHAR(1000) = N'How do I track my order?';
    ```

1. Run the script again. 

1. Review the updated outputs: 
  - retrieved context 
  - grounded prompt 
  - AI answer 

### Test an unsupported question 

1. Try a question that may not exist in the FAQ: 

	
sql
    DECLARE @user_question NVARCHAR(1000) = N'Can I pay using cryptocurrency?';
    
 

  ![x1bkjb6g.png](../../media/x1bkjb6g.png) 

1. Run the script again. 

1. Review the results. 

    - If the FAQ does not contain relevant information, the model should respond with something similar to: 

	
nocode
    I do not know. 
    ```

    ![l4yx9ngm.png](../../media/l4yx9ngm.png) 

This demonstrates how **RAG grounding helps reduce hallucinations**. 

--- 

# What you just built 

You implemented a complete **Retrieval-Augmented Generation workflow using Azure SQL**. 


nocode
User question 
      ↓ 
Azure SQL retrieval 
      ↓ 
Top FAQ rows 
      ↓ 
Context assembly  
      ↓ 
Grounded prompt 
      ↓ 
GPT-4o generation 
      ↓ 
Grounded AI answer 


You've completed this exercise. Select **Next** to continue. 

=== 

# Exercise 4 - Orchestrate the AI FAQ workflow with Microsoft Foundry Agents 

## Learning objective 

In this exercise, you'll use **Microsoft Foundry Agents** to orchestrate the FAQ workflow end-to-end. 

You'll configure an agent that can: 

- Receive a natural language support question. 
- Call an **MCP tool**. 
- Use that tool to retrieve FAQ matches from your service. 
- Ground the response using approved FAQ content. 
- Return a final answer to the user. 

For this exercise, the MCP tool is running **locally** on your machine and is exposed securely to Foundry by using **dev tunnel**. 

By the end of this exercise, you'll understand how to connect **Microsoft Foundry Agents** to external tools through **Model Context Protocol (MCP)** and use the agent to orchestrate retrieval-based FAQ answering. 

Conceptually, the flow looks like this: 


-nocode
User question 
      ↓ 
Microsoft Foundry Agent 
      ↓ 
MCP tool call 
      ↓ 
Local MCP server 
      ↓ 
FAQ retrieval / business logic 
      ↓ 
Tool result returned to agent 
      ↓ 
Grounded final response 


--- 

## Scenario 

So far, you've: 

- Stored FAQ content in Azure SQL Hyperscale. 
- Used vector search to retrieve relevant FAQ entries. 
- Built a grounded prompt for GPT-4o. 

Now you'll move the orchestration layer into **Microsoft Foundry Agents**. 

Instead of manually running SQL scripts, the agent will decide when to call the MCP tool, retrieve the relevant FAQ content, and generate a grounded response. 

This gives you a more realistic agent-based architecture for support assistant scenarios. 

--- 

## Part 1 - Start the local MCP tool 

## Steps 

1. In **VS Code**, select **Ctrl + Shift + E** to open **Explorer**. 

1. Select **Open Folder** and navigate to C:\LabFiles\sql_mcp_server. 

    - Select **Select folder**. 
    - Select **Don't Save**. 
    - Select **Yes, I trust the authors**. 

   ![be1pqu23.png](../../media/be1pqu23.png) 

1. Open a **new terminal** in VS Code. 

    - Select **Terminal** from the top menu and select **New Terminal**. 

1. Create a Python virtual environment: 

	
PowerShell
    python -m venv .venv
    ```

	
PowerShell
    .\.venv\Scripts\Activate.ps1
    ```

1. Install required dependencies: 

	
PowerShell
    pip install -r requirements.txt
    ```

1. Start the MCP server: 

	
PowerShell
    python server.py
    ```

	> Select **Allow** on the **Windows Security** pop-up window. 

1. Verify the server is running. 

    - You should see output similar to: 

	
nocode
    [MCP] Starting FAQ SQL Assistant on http://0.0.0.0:8000 
    [MCP] MCP endpoint : http://0.0.0.0:8000/mcp 
    ```

	> Keep this terminal running. 

--- 

## Part 2 - Expose the local MCP server with dev tunnel 

Because Microsoft Foundry must be able to reach your local MCP server, you'll expose it using **dev tunnel**. 

## Steps 

1. Open a new terminal window. 

1. Run the following command:

	
-bash
    deactivate
	```

    >[!note]Deactivating a virtual environment returns your terminal to the system Python, so commands use global packages instead of the project's isolated environment.

1. Start a dev tunnel that forwards traffic to your local MCP server port. 

    - Run the following command to sign in: 

	
bash
    devtunnel user login
    ```

    - Select **Work or school account**.
    - Sign in with your Microsoft Entra account ID:  
    **Username**: `@lab.CloudPortalCredential(User1).Username`  
    **TAP**: `@lab.CloudPortalCredential(User1).AccessToken` 

    - Select **Yes**.
    - Select **Done**.
    - Run the following commands to complete the setup: 

	
bash
    devtunnel create my-faq-tunnel@lab.LabInstance.Id --allow-anonymous
    ```

	
bash
    devtunnel port create my-faq-tunnel@lab.LabInstance.Id -p 8000 --protocol http
    ```

	
bash
    devtunnel host my-faq-tunnel@lab.LabInstance.Id
    ```

1. Review the output. 

    - You should receive a public HTTPS forwarding URL similar to: 

	
-nocode
    https://<your-tunnel-name>.devtunnels.ms:8000
    ```

    ![lduht6k7.png](../../media/lduht6k7.png) 

1. Copy the public tunnel URL into the field below: 

    @lab.TextBox(tunnelURL) 

	> This URL will be used when configuring the MCP tool connection in Foundry. 

    ![nrqdjatq.png](../../media/nrqdjatq.png) 

	> Keep the dev tunnel running during the exercise. 

--- 

## Part 3 - Add the MCP tool to the Foundry Agent 

## Steps 

1. Open Edge and go to `https://ai.azure.com/`. 

    - Select **Sign In** from the upper-right. 

1. Select the **New Foundry** slider from the upper-right of the page. 

1. Select **FAQ-Assistant-project** and select **Let's go**. 

1. Select **Build** from the menu ribbon in the upper-right of the page. 

	![s6g55sfe.png](../../media/s6g55sfe.png)

1. Select **Tools** from the menu on the left side of the page. 

1. Select the **Tools** tab, then select **Connect a tool**. 

1. In the **Select a tool** screen: 

  - Select the **Custom** tab. 
  - Choose **Model Context Protocol (MCP)**. 
  - Select **Create**. 
  
  ![0huyarqj.png](../../media/0huyarqj.png)

1. Configure the connection with the attributes below, then select **Connect**: 
	
    | Setting | Value |
    | -------- | -------- |
    | Name    | `faq@lab.LabInstance.Id`         |
    | Remote MCP Server endpoint    | `@lab.Variable(tunnelURL)/mcp`         |
    | Authentication    | Unauthenticated         |

1. Create a new agent. 

    - Select **Use in an agent**. 
    - Enter `faq-orchestrator-agent` for the **Agent name**. 
    - Select **Create and open playground**. 

1. Configure the agent instructions. 

    In the **Instructions** card, provide guidance similar to the following: 

	
text
    You are a support FAQ assistant. 

    Use the available MCP tool to retrieve relevant FAQ content before answering customer questions. 

    Answer using ONLY the tool results when possible. 
    If the tool does not return relevant information, say that you do not know based on the available FAQ content. 

    Do not invent policies, refunds, shipping details, or support actions that are not present in the tool output. 
    ```

1. Remove **Web search** from the **Tools** list.

	![rrheryw9.png](../../media/rrheryw9.png)

1. Confirm that Foundry can discover the tool definitions exposed by your MCP server. 

    - You should see the available tool or tools listed in the agent tool configuration.
    
    ![f8zc3d2m.png](../../media/f8zc3d2m.png) 

1. Select **Save** in the upper-right to save the agent configuration. 

--- 

## Part 4 - Test the agent end-to-end 

## Steps 

1. Open the agent chat interface in Foundry. 

1. Submit a support question. 

    - Example: 

	
chat
    My product arrived damaged.
    ```

    >[!note]If you see this notification, select **Always approve all tools**.   
    	![87605nmz.png](../../media/87605nmz.png)

1. Review the tool activity. 

    - You should see the agent invoke the MCP tool before producing a final answer. 

1. Review the final answer returned by the agent. 

    - Expected behavior: 

		- The answer is based on retrieved FAQ content. 
        - The answer stays within the approved support knowledge. 
        - The response does not invent unsupported details. 

    - Example expected grounding behavior: 

      - The tool may return a FAQ such as **How do I return a damaged item?** 
      - The agent should summarize that information into a natural response 

--- 

## Try another example 

1. Ask a second question: 

	
chat
    Where can I check my delivery status?
    ```

1. Review the result. 

    - The agent should call the MCP tool and return an answer grounded in the FAQ content. 
    - You should see behavior similar to: 

	
-nocode
    How do I track my order?
    ```

--- 

## Test an unsupported question 

1. Ask a question that may not be covered by the FAQ data. 

    - Example: 

	
chat
    Can I pay using cryptocurrency?
    ```

1. Review the answer. 

    - Expected behavior: 

      - The agent may still call the tool. 
      - If no relevant FAQ content is returned, the agent should respond with a grounded fallback such as: 

		
-nocode-wrap
        I do not know based on the available FAQ content. 
        ```

	> This shows that the agent is using retrieval and tool results rather than hallucinating an answer. 

1. Return to **VS Code** and open the terminal window running the Python server from Part 1. 

    - Stop the server by selecting **Ctrl+C**. 

You've completed this exercise. Select **Next** to continue. 

=== 

# Exercise 5 - Integrate Azure SQL Hyperscale with Microsoft Fabric for Analytics 

## Learning objective 

In this exercise, you'll enable **real-time analytics** by mirroring Azure SQL Hyperscale data into **Microsoft Fabric** without building ingestion pipelines. 

You'll: 

* Enable **Mirroring for Azure SQL Database** in Microsoft Fabric. 
* Selectively mirror supported tables into **OneLake**. 
* Query mirrored data using the **SQL Analytics Endpoint**. 
* Create a **semantic model**. 
* Build a **Power BI report** on mirrored data. 
* Explore **data lineage** across Fabric assets. 

By the end of this exercise, you'll understand how Fabric enables analytics on operational data with minimal engineering effort. 

--- 

## Scenario 

Your Azure SQL Hyperscale database currently supports two key workloads: 

- **Transactional + AI workload** 
  - FAQ content 
  - Vector embeddings for semantic search 

- **Analytics workload** 
  - Reporting on FAQs 
  - Category insights 
  - Content analysis 

Instead of building ETL pipelines, you'll use **Microsoft Fabric Mirroring** to replicate only the **analytics-ready data** into **OneLake**. 

--- 

## Architecture overview 


nocode-wrap
Azure SQL Hyperscale 
 └── FAQ_Content (Operational Data) 

        ↓
Fabric Mirroring (Near Real-Time with selective Replication) 

        ↓ 
OneLake (Delta Tables) 

        ↓ 
SQL Analytics Endpoint 

        ↓ 
Semantic Model 

        ↓ 
Power BI Report 


--- 

## Part 1 - Enable mirroring for Azure SQL Hyperscale 

## Steps 

1. Open **Microsoft Fabric** in the browser. 

    `https://app.fabric.microsoft.com` 

1. Select **My workspace** from the left menu and create a new workspace. 

  - Select **+ New workspace**. 
  - Name the workspace `Workspace@lab.LabInstance.Id`. 
  - Select **Apply**. 

1. From the workspace, select **+ New Item**. 

1. Search for and select `Mirrored Azure SQL Database`. 

   ![bl7ci1m2.png](../../media/bl7ci1m2.png) 

1. Select **Azure SQL Database**. 

   ![pd05b21b.png](../../media/pd05b21b.png) 

1. Enter connection details: 

 

  | Setting                       | Value                                                       | 
  | ------------------------------| ----------------------------------------------------------- | 
  | **Server name**               | `faq-ai-assistant-@lab.LabInstance.Id.database.windows.net` | 
  | **Database**                  | `faq-ai-assistant-db`                                       | 
  | **Authentication kind**       | **Basic**                                                   | 
  | **Username**                  | `admin-@lab.LabInstance.Id`                                 | 
  | **Password**                  | `@lab.CloudPortalCredential(User1).Password`                | 
  | **Privacy Level**             | **None**                                                    | 
  | **Use encrypted connection**  | **Enabled**                                                 | 

	> Basic Authentication corresponds to **SQL Authentication** 

  ![cm2qq90s.png](../../media/cm2qq90s.png) 

1. Select **Connect**. 

1. In the **Choose data** screen: 

   - Available tables: 

     - `dbo.FAQ_Content` 
     - `dbo.FAQ_Embeddings` 

   **Select ONLY:** 

    - `dbo.FAQ_Content` 

   **Do NOT select:** 

    - `dbo.FAQ_Embeddings` 

   ![istdwnc2.png](../../media/istdwnc2.png) 

	>[!note] FAQ_Embeddings contains a **vector column**. Vector data types are **not supported in Fabric Mirroring**. 

1. Select **Connect**. 

1. Confirm destination settings: 

    - Name: `faq-ai-assistant-db` 

1. Select **Create mirrored database**. 

  ![v6h454n7.png](../../media/v6h454n7.png) 

## Expected result 

- Fabric connects to Azure SQL Hyperscale 

- Only **FAQ_Content** is selected and begins syncing 

- Mirrored data is stored in **OneLake** 

- No ETL pipelines are required 

--- 

## Part 2 - Verify data in OneLake 

1. Open the mirrored database item. 

1. Navigate to **Tables in OneLake**. 

1. Refresh the tables and confirm the availability of **FAQ_Content**. 

    ![j7vlxlgn.png](../../media/j7vlxlgn.png) 

	>[!alert] The replication process may take a few minutes. Select **Refresh** under the **Monitor replication** section and wait until it shows 15 rows replicated.
    ![yvis2ldu.png](../../media/yvis2ldu.png)

1. In the upper-right area of the mirrored database page, select the **Mirrored database** dropdown menu and select **SQL analytics endpoint**.  

  ![hl3k45zd.png](../../media/hl3k45zd.png) 

	>[!note] The SQL analytics endpoint provides a read-only query surface over the mirrored data. 

1. In the **Explorer** pane, expand the mirrored database objects and locate **dbo.FAQ_Content**. 

    ![9kcskhos.png](../../media/9kcskhos.png) 

1. Select **New SQL query** at the top of the page, enter the following query, and select **Run**: 

	
sql
    SELECT TOP 10 *
    FROM FAQ_Content
    ```

  ![jsbnd9uk.png](../../media/jsbnd9uk.png) 

--- 

## Part 3 - Create a Power BI semantic model and report 

1. Return to the mirrored database item in your workspace. 

    ![dkmobp8l.png](../../media/dkmobp8l.png) 

1. On the mirrored database page, select **Create semantic model** and configure the following settings: 

  | Setting                       | Value                                                       | 
  | ------------------------------| ----------------------------------------------------------- | 
  | **Semantic model name**       | `FAQ_Content`                                               | 
  | **Workspace**                 | **Workspace@lab.LabInstance.Id**                            | 
  | **Storage mode**              | **Direct Lake on SQL**                                                                          | 
  | **Tables**                    | **FAQ_Content**                                             | 

  ![o9uffess.png](../../media/o9uffess.png) 

	>[!note] This creates a Power BI semantic model based on the mirrored database. 

1. Select **Confirm** and wait for the semantic model creation process to finish. 

1. In the left pane, select the **Workspace@lab.LabInstance.Id** workspace, then select it again from the menu.

1. In the list of resources, select the ellipses (…) next to the **FAQ_Content** semantic model and select **Create report**. 

    ![8nh95eug.png](../../media/8nh95eug.png) 

1. On the command bar at the top, select **Copilot**. 

1. In the Copilot chat, submit the following prompt:  

	
chat
    What's in my data?
    ```

1. Review the response. 

1. Submit the following prompt and review the response: 

	
chat
    FAQ count by category
    ```

1. Select **File** from the upper-left menu, then select **Save**. 

    - Name the file `FAQ_rpt` and then select **Save**. 

--- 

## Part 4 - Explore lineage 

1. Return to **Workspace@lab.LabInstance.Id** and select **Lineage view** from the upper-right menu. 

    ![3m8blx72.png](../../media/3m8blx72.png) 

1. Review the data flow between components. 

    - You should see relationships such as: 

	
-nocode
    Azure SQL Database 
            ↓ 
    Mirrored Database 
            ↓ 
    SQL Analytics Endpoint / Semantic Model 
            ↓ 
    Power BI Report 
    ```

 ![4tdd0o9k.png](../../media/4tdd0o9k.png) 

--- 

## Part 5 - Understand the split architecture 

At this point, your system follows a **modern enterprise pattern**: 

### AI / RAG Layer 


-nocode
Azure SQL 
 → FAQ_Content 
 → FAQ_Embeddings (VECTOR) 
 → Used by Foundry Agents + MCP 
 ```

### Analytics Layer 


-nocode
Fabric 
 → Mirrored FAQ_Content 
 → Lakehouse / Warehouse 
 → Power BI 
 ```

--- 

## Why this matters 

This separation allows: 

- Optimized performance for each workload 
- No duplication of logic 
- Minimal data engineering effort 

--- 

## What you just built 

You implemented **real-time analytics** on top of Azure SQL using Microsoft Fabric, without building pipelines. 


-nocode
Azure SQL Hyperscale (Operational + AI) 
        ↓ 
Fabric Mirroring (No ETL) 
        ↓ 
OneLake (Auto Synced) 
        ↓ 
Lakehouse / Warehouse 
        ↓ 
Power BI Analytics 


You've completed this exercise. Select **Next** to continue. 

=== 

# Exercise 6 - Expose Azure SQL Hyperscale using SQL MCP server (Data API builder) 

## Learning objective 

In this exercise, you'll use **SQL MCP Server** built into **Data API builder (DAB)** to expose Azure SQL Hyperscale as an MCP-compatible tool. 

You'll: 

- Configure Data API builder for Azure SQL Hyperscale. 

- Use a **prebuilt DAB configuration**. 

- Use a **prebuilt MCP configuration for VS Code**. 

- Start SQL MCP Server locally. 

- Connect to it from Visual Studio Code. 

- Query your database through MCP. 

By the end of this exercise, you'll understand how to expose Azure SQL Hyperscale to AI agents using a **standardized, production-ready MCP layer**. 

--- 

## Architecture overview 


-nocode
Azure SQL Hyperscale 
        ↓ 
Data API builder (DAB) 
        ↓ 
SQL MCP Server 
        ↓ 
VS Code (Copilot Chat) 
        ↓ 
AI grounded on SQL data 


--- 

# Part 1 - Install Data API builder 

## Steps 

1. Return to **VS Code** and open a new terminal window.

1. Run the following commands to create a working folder: 

	
PowerShell
    mkdir C:\LabFiles\sql-mcp-lab
    ```

	
PowerShell
    cd C:\LabFiles\sql-mcp-lab
    ```

1. Initialize tool manifest: 

	
PowerShell
    dotnet new tool-manifest
    ```

1. Install DAB: 

	
PowerShell
    dotnet tool install --local Microsoft.DataApiBuilder --version 1.7.90
    ```

1. Verify: 

	
PowerShell
    dotnet tool run dab --version
    ```

	> Ensure version is **1.7 or later**. 

--- 

# Part 2 - Create prebuilt DAB configuration 

1. In VS Code, select **File** > **Open Folder**.

1. Navigate to C:\LabFiles\sql-mcp-lab and select **Select folder**.

	- Select **Yes, I trust the authors**.
    

1. In VS Code, open a new terminal window.

1. Create a new file in the sql-mcp-lab folder: 

	
PowerShell
    code dab-config.json
    ```

    > This will open the new file in the VS Code editor.

1. Paste the following configuration in the **dab-config.json** file: 

	
json
    {
      "$schema": "https://github.com/Azure/data-api-builder/releases/download/v1.7.90/dab.draft.schema.json",
      "data-source": {
        "database-type": "mssql",
        "connection-string": "Server=tcp:faq-ai-assistant-@lab.LabInstance.Id.database.windows.net,1433;Initial Catalog=faq-ai-assistant-db;Persist Security Info=False;User ID=@lab.CloudResourceTemplate(AzureHyperscaleSQL).Parameters[SQLAdminUsername];Password=@lab.CloudResourceTemplate(AzureHyperscaleSQL).Parameters[sqlAdministratorLoginPassword];MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
      },
      "runtime": {
        "rest": {
          "enabled": true
        },
        "mcp": {
          "enabled": true
        },
        "host": {
          "mode": "development"
        }
      },
      "entities": {
        "faqContent": {
          "source": {
            "object": "dbo.FAQ_Content",
            "type": "table"
          },
            "primary_key": [ "Id" ],
          "permissions": [
            {
              "role": "anonymous",
              "actions": [ "read" ]
            }
          ],
          "mcp": {
            "description": "Approved FAQ content used to answer customer support questions."
          }
        }
      }
    }
    ```

    - Save the file by selecting **Ctrl+S**. 

	>[!note] What this config does: 
	> 
    - Connects to **Azure SQL Hyperscale**.  
    - Exposes **FAQ_Content** as MCP-readable entity.  
    - Enables MCP server inside DAB.  
    - Restricts access to **read-only (safe for AI)**. 

--- 

# Part 3 - Add prebuilt MCP configuration for VS Code 

1. In the terminal, run the following to create the **.vscode** folder and **mcp.json** file: 

	
PowerShell
    mkdir .vscode; code .vscode\mcp.json
    ```

1. Paste this configuration into the **mcp.json** file: 

	
json
    {
      "servers": {
        "sql-mcp-server": {
          "type": "stdio",
          "command": "dotnet",
          "args": [
            "tool",
            "run",
            "dab",
            "start",
            "--mcp-stdio",
            "role:anonymous",
            "--LogLevel",
            "None",
            "--config",
            "${workspaceFolder}/dab-config.json"
          ]
        }
      }
    }
    ```

    - Save the file by selecting **Ctrl+S**. 

	>[!note] What this does:  
	> 
    - Launches **SQL MCP Server** automatically  
    - Connects VS Code → DAB → Azure SQL Hyperscale  
    - No manual server wiring needed 

--- 

## Part 4 - Discover MCP tools 

1. In the VS Code chat pane, select **Configure Tools**.

    ![a0llhtb2.png](../../media/a0llhtb2.png)

1. In the **Configure Tools** pane, ensure the **sql-mcp-server** and its tools are showing.

	![e7zj3agu.png](../../media/e7zj3agu.png)

    > You may need to select **Update Tools** below it. The tool update may take a minute or two.

--- 

# Part 5 - Query Azure SQL via MCP 

1. Ask: 

	
chat
    By leveraging MCP tool - Find the number of database records in FAQ_Content table.
    ```

	>[!note]If you see this notification, select **Allow in this Session**. 
        ![i0yt61qd.png](../../media/i0yt61qd.png) 

    - Expected Behavior:  
        - VS Code → MCP  
		- MCP → Azure SQL Hyperscale  
        - Returns FAQ rows 
        - Response grounded in database 

1. Try the additional query below: 

	
chat
    Find FAQ entries related to damaged products. 
    Show me the different type of categories that exist in the faqcontent table. 
    ```

--- 

# Troubleshooting 

If it doesn't work: 

- Ensure DAB is running. 
- Check **dab-config.json** connection string. 
- Restart VS Code. 
- Confirm Copilot Chat is enabled. 
- Ensure DAB version ≥ 1.7 

--- 

# What you learned 

| Component            | Role            | 
| -------------------- | --------------- | 
| Azure SQL Hyperscale | Data layer      | 
| Data API builder     | API + MCP layer | 
| SQL MCP Server       | Tool interface  | 
| VS Code              | MCP client      | 

--- 

# Why this matters 

This approach: 

- Removes need for custom MCP servers 
- Uses Microsoft-native tooling 
- Enables secure AI access to SQL 
- Scales to enterprise workloads 

--- 

# Final architecture 


-nocode
Azure SQL Hyperscale 
        ↓ 
Data API builder 
        ↓ 
SQL MCP Server 
        ↓ 
VS Code / Copilot 
        ↓ 
Grounded AI responses 


--- 

# Key Takeaway 

You used **SQL MCP Server (Data API builder)** to turn Azure SQL Hyperscale into an **AI-ready, MCP-compatible data service** without writing custom APIs. 

--- 

 
