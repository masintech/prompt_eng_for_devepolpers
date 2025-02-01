# This is from the course ChatGPT Prompt Engineering for Developers

[ChatGPT Prompt Engineering for Developers](https://learn.deeplearning.ai/courses/chatgpt-prompt-eng)


1. Using gpt-4o-mini instead of gpt3.5-turbo
2. Also experiment on deepseek-r1:latest 7b.


# Types of prompt engineering techinique

## The Flow of this lecture
1. Load the model through open AI API
2. Using a basic function to get a completion response from **user prompt**

```python
  def get_completion(prompt, model="gpt-4o-mini"):
    messages = [{"role": "user", "content": prompt}]
    response = openai.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0, # this is the degree of randomness of the model's output
    )
    return response.choices[0].message.content 

```
user prompt using f string in pyhton.
## Iteration

Basically, try and error to get the desired answers:
1. Idea
2. Implementation
3. Experimental Result
4. Error Analysis
### Case: generating marketing product description from a product fact sheet (technical spec)

prompt:
delimited by triple backticks

```python
f"""
Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.
Technical specifications: `{fact_sheet_chair}` """
```

The issues:
1. The text is too long
	 - ask the AI to "Use at most 50 words"
 2. Text focuses on the wrong details
      - add more descriptions
         "The description is intended for furniture retailers, 
		so should be technical in nature and focus on the 
		materials the product is constructed from.
		At the end of the description, include every 7-character 
		Product ID in the technical specification. "
   3. Print in HTML
	   "Format everything as HTML that can be used in a website. 
	   Place the description in a <div> element."
		`from IPython.display import display, HTML`
		`display(HTML(response))`

## Summarizing
### Case: Short summary of a product review
    1. word/sentence/character limit
    2. focus on shipping and delivery
    3. focus on price and value
    4. multiple product reviews using python loops
    ```python
    reviews = [review_1, review_2, review_3, review_4]
    for i in range(len(reviews)):
    prompt = f"""
    Your task is to generate a short summary of a product \ 
    review from an ecommerce site. 

    Summarize the review below, delimited by triple \
    backticks in at most 20 words. 

    Review: ```{reviews[i]}```
    """

    response = get_completion(prompt)
    display(Markdown(i, response, "\n"))
    ```

## Inferring
### Case Product Review Text
1. Identify type of emotions (sentiments)
2. Extract product and company name from customer reviews
3. Doing multiple tasks at once, instructing AI to use JSON format
```python
prompt = f """
Format your response as a JSON object with \
"Sentiment", "Anger", "Item" and "Brand" as the keys.
If the information isn't present, use "unknown" \
as the value.
Make your response as short as possible.
Format the Anger value as a boolean.
"""
```
```json 
 { "Sentiment": "positive", "Anger": false, "Item": "lamp", "Brand": "Lumina" } 
 ``` 
4. Identify multiple topics: make a news alert
```python
prompt = f"""
Determine whether each item in the following list of \
topics is a topic in the text below, which
is delimited with triple backticks.

Give your answer as follows:
item from the list: 0 or 1
"""

List of topics: {", ".join(topic_list)}
```
Output
	nasa: 1  
	local government: 0  
	engineering: 0  
	employee satisfaction: 1  
	federal government: 0	

```python
topic_dict = {i.split(': ')[0]: int(i.split(': ')[1]) for i in response.split(sep='\n')}
if topic_dict['nasa'] == 1:
    print("ALERT: New NASA story!")
```


## Transforming
1. language translation
2. grammar/spell  check
3. proofread: make it more compelling, APA style and targets on advanced reader etc.

## Expansion
### Case: auto reply a customer mail with given customer's sentiment
**Specify in the prompt**
```python
prompt =  f"""
If the sentiment is negative, apologize and suggest that \
they can reach out to customer service. 
Make sure to use specific details from the review.
Write in a concise and professional tone.
Sign the email as `AI customer agent`.
"""
```
Using **Temperature** parameter to tune make the response various, so the higher the temperature the more flexible it is.

## Chatbot

### Models of chatbot

1. system: command assistant
2. assistant: interact with user
3. user: interact with assistant

Giving the context for the AI, so that the assistant can answer the questions accordingly.
Using prompt with message with all the roles.
```python
def get_completion_from_messages(messages, model=MODEL, temperature=0):
    response = openai.chat.completions.create(
        model=model,
        messages=messages,
        temperature=temperature, # this is the degree of randomness of the model's output
    )
    # print(str(response.choices[0].message))
    return response.choices[0].message.content
```
Message could be 
```python
messages =  [  
{'role':'system', 'content':'You are an assistant that speaks like Shakespeare.'},    
{'role':'user', 'content':'tell me a joke'},   
{'role':'assistant', 'content':'Why did the chicken cross the road'},   
{'role':'user', 'content':'I don\'t know'}  ]
```

We could see the answer is by assistant while printing response.choices[0].message
```python
response = get_completion_from_messages(messages, temperature=1)

ChatCompletionMessage(content='Ah, good sir or fair lady, the jest doth lie in the answer straight and true! The chicken crossed the road to reach the other side. A simple riddle, yet it doth hold a warm jest within its folds. Wouldst thou care for another mirthful quip?', refusal=None, role='assistant', audio=None, function_call=None, tool_calls=None)
```