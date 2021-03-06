### What does this do?

The degradation tester sends a list of Chatbot designer specified questions and their labelled intents to the Watson bot and return it's intent and confidence. This is then compared to ensure that there are no surprises or negative impacts when pushing to production. It's a tool to cut down on time spent testng by utilising a standard dataset defined by each user to ensure there are no unintended consquences or confusion added when building new content.

The write up Github focuses on the implementation and setup of Analyser, for a writeup on the metrics please see: [https://callumkemp.medium.com/ibm-watson-assistant-model-monitoring-and-improvements-4e39c2d10560](https://callumkemp.medium.com/ibm-watson-assistant-model-monitoring-and-improvements-4e39c2d10560)

### Why use this?

1. Easily track changes to control questions confidence intended or otherwise
2. Testing in bulk rather than manually saving time and being accurate.
3. Ensure new question example haven't stopped old questions from being understood
4. Test questions without customers being impacted

### File setup

There are two CSV files that are read and written to enable this to work. One is a list of questions and previous scores per question and the other is a log of headline results to compare when and where something has gone wrong. These files will be updated in both the Python and Notebook files. The Python file will take the input and send it to your Watson Assistant.

This uses the IBM Assistant V2 API [https://cloud.ibm.com/apidocs/assistant/assistant-v2?code=python](https://cloud.ibm.com/apidocs/assistant/assistant-v2?code=python)

You will need your API Key, URL endpoint and Assistant ID to use make requests.

### You will need to make sure that you connect your skill to an Assistant to be able to run this.

![https://i.ibb.co/G5F09WN/Screenshot-2020-09-18-at-17-32-12.png](https://i.ibb.co/G5F09WN/Screenshot-2020-09-18-at-17-32-12.png)

### Best practice for your dataset of questions.

- Do not use word for word matches into your intents these will just always return a score of 1 regardless.
- Use real customer questions where possible. Questions that came directly from customers are the best form of data to test against.
- Use questions that are similar to ones users are likely to ask.
- Provide at least one example per intent if you don't you may experience issues with the confusion matrix. 5 or more per intent is better.
- Try to provide a good coverage for customer questions i.e long questions / short questions and ones that are relatively poorly worded to get the most out of testing.
- Make the number of examples per intent match how popular that intent is for your customers to ask if 10% of your customers are asking about mortgage rates have 10% of your dataset labelled as your mortgage rates intent.

You will only need to make changes to column A and B add your example question that you'd like to test and the intent that question should trigger in column b.

![https://i.ibb.co/nb9J1Ky/Screenshot-2020-09-21-at-00-28-16.png](https://i.ibb.co/nb9J1Ky/Screenshot-2020-09-21-at-00-28-16.png)

### Files included

- Python Notebook
- Python File
- Example Log File
- Example Question Set
- Example Watson Skill
- Example Confusion Matrix

### Customisation to Watson Skill required

This implementation requires you to add one node to your Workspace in order to work. The reason for this is if you access the system provided intent and entities anything below 20% will not return a result. By logging the information inside of Watson as a context and then accessing it as part of the payload it will allow you to see which intent something low confidence is leaning towards. If you’d like to use the system only confidence please see the cusotmisation instructions.

If you're still unsure about this please upload the skill provided to your IBM cloud account and add it to your assistant to see how it works.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f123e425-6487-432d-898d-60287c7649de/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f123e425-6487-432d-898d-60287c7649de/Untitled.png)

The message is sent with the context $ConfidenceTester which will trigger the node we want to test and log the context. In the sender function we also specify return context. If this isn't sent as part of the request there will be nothing returned to be logged. This is done specifically so that the confidence score and intent are returned rather than a low confidence message.

```
input={
        'message_type': 'text',
        'text': x,
     'options': {
            'return_context': True
        }
    },
    context={
        'skills': {
            'main skill': {
                'user_defined': {
                    'ConfidenceTester': 'Yes'
                }
            }
        }
    }
    }

```

### Confusion Matrix

A confusion matrix is a great way to check labelled data predictions vs their actual results when there's only a handful of predictions. The results are normalised so if you have 30 examples of intent X and only 2 Examples of intent Y you will get a consitent set of results.

The below shows a solid red running through the matrix showing the prediction matches the output. This file is overwritten each time it's created if you'd like to keep a historic version you can move it to another folder or rename the file after it's saved.

To change the styling please see the Seaborn documentation [https://seaborn.pydata.org/generated/seaborn.heatmap.html](https://seaborn.pydata.org/generated/seaborn.heatmap.html)

More information can be found [https://en.wikipedia.org/wiki/Confusion_matrix](https://en.wikipedia.org/wiki/Confusion_matrix)

![https://i.ibb.co/pK9QPcc/Confusion-Matrix.png](https://i.ibb.co/pK9QPcc/Confusion-Matrix.png)

### Customisation

### Log the standard system confidence

If you'd like to just log the output from Watson including the lower than 20% irrelevant results you can do that be replacing the below code

```
    IntentList.append((response['context']['skills']['main skill']['user_defined']['Intent']['intent']))
    ConfidenceList.append((response['context']['skills']['main skill']['user_defined']['Intent']['confidence']))

```

with

```
 try:
        ConfidenceList.append(response['output']['intents'][0]['confidence'])
    except:
        ConfidenceList.append(0)
    try:
        IntentList.append(response['output']['intents'][0]['intent'])
    except:
        IntentList.append("ZLowConf")

```

ZLowConf is used to keep the confusion matrix looking clean. You can call this anything you want if you don't use the Confusion Matrix

## Confidence Average across Intents.

This box and whisker shows the averages and interquartile ranges of intents distributions.

![https://i.ibb.co/vJ6Cw6J/Untitled-1.png](https://i.ibb.co/vJ6Cw6J/Untitled-1.png)




## How well defined an intent is.

Analysing the score of the difference between the highest scoring intent and it's closest competi

![https://i.ibb.co/G3Zt8JJ/Untitled-2.png](https://i.ibb.co/G3Zt8JJ/Untitled-2.png)


I’ve used the content from the Watson content catalogue for Mortgages for testing and example purposes but this can easily be customised to your own question set by exporting your intents as a CSV and then copying them in and making changes.
