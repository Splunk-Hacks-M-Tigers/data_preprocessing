# Data preprocessing

The primary goal of this Python script is to transform a raw, unlabeled or minimally labeled dataset (specifically, the xTRam1/safe-guard-prompt-injection dataset from Hugging Face) into a structured CSV format that is optimized for Splunk's data ingestion capabilities and the Splunk MLTK's requirements for machine learning model training.

This transformation involves:

 - Standardizing the timestamp (_time): Ensuring a consistent and Splunk-compatible timestamp format for proper indexing.


 - Creating a clear label field: Converting the target variable into a numerical format (0 for benign, 1 for malicious/injection) that MLTK algorithms can use.


 - Augmenting with "CIM-friendly" contextual fields: Adding common fields typically found in security logs (app, user, src_ip, severity, etc.) to enhance the dataset's realism and utility within a Splunk environment, even if they are generated with illustrative values for a proof-of-concept.


 - Maintaining the raw text field: Preserving the original prompt content, which will serve as the primary feature for text-based analysis in MLTK.


 - Separating training and testing data: Ensuring distinct datasets for model training and unbiased evaluation.

## Column Selection and Rationale

The script strategically selects and generates specific columns to facilitate prompt injection detection in Splunk:

- _time (Timestamp):

  - Why it's chosen: This is a fundamental field in Splunk. Every event in Splunk must have a timestamp. By pre-generating it in a consistent ISO 8601 format (YYYY-MM-DDTHH:MM:SS.000Z), we ensure Splunk correctly indexes and orders the events, which is crucial for any time-based analysis or correlation.

  - How it's generated: Random dates within a reasonable recent range (e.g., last 60 days) are assigned to each event. While not reflecting the exact real-world timestamp of a specific prompt, this ensures a valid time distribution for Splunk's indexing.


- text (Prompt Content):

    - Why it's chosen: This is the core data for prompt injection detection. It contains the actual user input to the LLM.

    - How it's used by MLTK: The raw text in this column will be transformed into numerical features using the TF-IDF (Term Frequency-Inverse Document Frequency) method directly within Splunk MLTK. TF-IDF will tokenize the text and convert words into numerical values that machine learning algorithms can process.


- label (Target Variable):

    - Why it's chosen: This is the ground truth indicating whether a prompt is a benign input (0) or a malicious injection (1). It's the target variable that the machine learning model will learn to predict.

    - How it's generated: Derived directly from the original dataset's label field, converted to an integer type.


- Contextual "CIM-Friendly" Fields (app, user, src_ip, session_id, message_id, severity, event_type, user_agent, http_method, url):

    - Why they are chosen: These fields are commonly found in application and security logs and align with the Splunk Common Information Model (CIM). While the primary detection happens on the text field, these provide invaluable context for:

      - Incident Investigation: If an injection is detected, these fields help identify which application was targeted, who the user was, their source IP address, the severity of the event, and other relevant details.

      - Dashboards and Reporting: Enable filtering, aggregation, and visualization of prompt injection events based on these contextual attributes in Splunk.

      - Future Feature Engineering: In a more advanced machine learning model, these fields (once properly normalized or encoded) could serve as additional predictive features alongside the text-based features, enhancing detection accuracy by incorporating behavioral patterns.

    - How they are generated (Illustrative for PoC): For this hackathon/proof-of-concept, these fields are generated with illustrative, dummy values (e.g., random IPs within private ranges, fixed application names, sequential IDs). This is a simplification. In a production environment, these would be extracted from real log sources and would contain actual, correlated data reflecting genuine user behavior and attack patterns.

