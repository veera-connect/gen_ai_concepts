# LangChain Conversation Memory --- Theory and Comparison

## 1. Introduction: Why does an AI application need memory?

A Large Language Model (LLM) generally responds using the information
available in the current request. In a multi-turn chatbot, a user
expects the assistant to understand references to earlier messages.

For example:

-   Human: My name is Kavya.
-   AI: Nice to meet you, Kavya.
-   Human: What is my name?
-   AI: Your name is Kavya.

To answer the last question correctly, the application must make
relevant earlier conversation information available to the model.
LangChain memory components help manage this conversation context.

**Important:** Conversation memory is not the same as changing the LLM's
trained knowledge. It manages information from the current conversation
and supplies relevant context when the application makes later requests.

This guide covers three memory types:

1.  `ConversationBufferMemory`
2.  `ConversationBufferWindowMemory`
3.  `ConversationSummaryMemory`

------------------------------------------------------------------------

## 2. ConversationBufferMemory

### What is it?

`ConversationBufferMemory` stores the complete conversation history
between the Human and the AI.

It keeps the messages from earlier turns and makes that history
available to the LLM during later interactions.

### Simple real-life example

Imagine a teacher maintaining a complete record of every question asked
by a student and every answer given by the teacher.

The teacher can refer to the entire discussion, including the first
question and the most recent question.

Conversation Buffer Memory works in a similar way.

### How it works

1.  The Human sends a message.
2.  The LLM generates a response.
3.  The Human message and AI response are added to the conversation
    history.
4.  When another message arrives, the previous history is made available
    to the LLM.
5.  The process continues, and the conversation history keeps growing.

### Example conversation

  -----------------------------------------------------------------------
  Turn                    Human                   AI
  ----------------------- ----------------------- -----------------------
  1                       My name is Kavya.       Nice to meet you,
                                                  Kavya.

  2                       I work as a Cloud       That sounds
                          Engineer.               interesting.

  3                       I am learning           Kubernetes is used to
                          Kubernetes.             manage containers.

  4                       What is my name?        Your name is Kavya.
  -----------------------------------------------------------------------

The memory retains all the exchanges, including Turn 1.

### Main characteristics

-   Stores the complete conversation history.
-   Retains both Human and AI messages.
-   Makes earlier messages available in later interactions.
-   The stored history grows as the conversation continues.
-   A longer history can require more context tokens when supplied to
    the LLM.

### Advantages

-   Earlier details remain available in the conversation history.
-   Useful when the chatbot needs to refer to a detailed discussion.
-   Easy to understand because the conversation is retained as a
    complete history.

### Limitations

-   Conversation history grows with every turn.
-   Longer history can increase token usage and request cost.
-   Very long conversations may approach the model's context limit.

### When should we use it?

Use Buffer Memory when retaining the full conversation is important and
the conversation is expected to remain reasonably short.

**One-line definition:** ConversationBufferMemory stores the complete
Human and AI conversation history and makes it available for future
interactions.

------------------------------------------------------------------------

## 3. ConversationBufferWindowMemory

### What is it?

`ConversationBufferWindowMemory` stores only the most recent **k
conversation turns**.

It maintains a limited window of conversation history. As new turns
arrive, older turns move outside the active window.

### What does k mean?

The parameter `k` determines how many recent conversation turns are
retained.

For example:

-   `k = 1`: retain the latest one conversation turn.
-   `k = 2`: retain the latest two conversation turns.
-   `k = 3`: retain the latest three conversation turns.

In a normal Human/AI exchange, one turn means one Human message and its
corresponding AI response.

### Simple real-life example

Imagine a teacher reviewing only the last two question-and-answer
exchanges with a student.

The teacher does not refer to the entire classroom discussion. The
teacher focuses on the latest two exchanges.

This is similar to Window Memory.

### How it works with k = 2

Imagine the following conversation:

  Turn   Human                         AI
  ------ ----------------------------- --------------------------
  1      My name is Kavya.             Nice to meet you.
  2      I work as a Cloud Engineer.   That sounds interesting.
  3      I am learning Kubernetes.     Good choice.

After Turn 3, the active memory window contains:

-   Turn 2: Cloud Engineer discussion.
-   Turn 3: Kubernetes discussion.

Turn 1 has moved outside the active window.

### Visual representation

``` text
Conversation timeline:

Turn 1          Turn 2          Turn 3
[ Older ]       [ Kept ]        [ Kept ]
                   <--- k = 2 --->
```

When another exchange is added, the window moves forward again. The
oldest exchange in the current window moves out.

### Main characteristics

-   Retains only the latest `k` conversation turns.
-   Older turns move outside the active memory window.
-   Keeps the amount of recent history limited.
-   A smaller window provides less historical context.
-   A larger window provides more historical context.

### Advantages

-   Controls how much recent conversation history is supplied.
-   Helps avoid continuously growing conversation context.
-   Useful for chatbots where recent messages matter more than older
    messages.

### Limitations

-   Older information may no longer be available in the active window.
-   The model may not be able to answer questions about details shared
    before the current window.
-   Choosing a very small `k` can remove useful context too quickly.

### When should we use it?

Use Window Memory when the latest conversation is more important than
the complete conversation history.

**One-line definition:** ConversationBufferWindowMemory retains only the
most recent `k` conversation turns and moves older turns outside the
active context.

------------------------------------------------------------------------

## 4. ConversationSummaryMemory

### What is it?

`ConversationSummaryMemory` uses an LLM to progressively summarize the
conversation.

Instead of keeping the entire conversation as a long sequence of
messages, it maintains a shorter summary containing important
information from the discussion.

The summary is updated as new conversation lines are added.

### Simple real-life example

Imagine a teacher having a long discussion with a student.

Instead of keeping every sentence in the student's discussion, the
teacher prepares short notes containing the important points.

When the student returns later, the teacher refers to those notes.

Conversation Summary Memory works in a similar way.

### How does progressive summarization work?

It follows four main steps:

1.  Start with the existing conversation summary.
2.  Collect the new lines of conversation.
3.  Ask the LLM to combine the existing summary with the new
    conversation lines.
4.  Create an updated summary.

This process continues as the conversation grows.

### Example of progressive summarization

**Initial conversation:**

-   Human: What do you think about Artificial Intelligence?
-   AI: Artificial Intelligence can be useful for people.

**Current summary:**

The human asks what the AI thinks about Artificial Intelligence. The AI
considers it useful for people.

**New lines of conversation:**

-   Human: Why do you think Artificial Intelligence is useful?
-   AI: It can help humans automate repetitive tasks and improve
    productivity.

**Updated summary:**

The human is interested in Artificial Intelligence. The AI considers it
useful because it can help people automate repetitive tasks and improve
productivity.

Notice that the new summary combines the earlier information and the new
information.

### Main characteristics

-   Uses an LLM to generate and update the summary.
-   Maintains a shorter representation of the conversation.
-   Combines the previous summary with new conversation lines.
-   Attempts to retain important details while reducing the amount of
    text.
-   The summary may not preserve every sentence or minor detail.

### Advantages

-   Helps manage long conversations with a condensed context.
-   Can retain important facts and discussion topics in a shorter form.
-   Avoids repeatedly supplying the entire raw conversation when the
    summary is used as the context.

### Limitations

-   Requires an LLM to create and update summaries.
-   Summarization may require additional model processing.
-   Important details may be omitted or simplified.
-   The summary may not preserve exact wording or every small detail.

### When should we use it?

Use Summary Memory when a conversation is long and retaining its
important information in a shorter form is more useful than keeping
every message in the active context.

**One-line definition:** ConversationSummaryMemory uses an LLM to
progressively summarize conversation history and maintain a concise
summary for future interactions.

------------------------------------------------------------------------

## 5. Difference between the three memory types

  ---------------------------------------------------------------------------------------------------------
  Feature           ConversationBufferMemory   ConversationBufferWindowMemory   ConversationSummaryMemory
  ----------------- -------------------------- -------------------------------- ---------------------------
  Main purpose      Store complete history     Store recent history             Maintain a concise summary

  Information       All conversation messages  Latest `k` turns                 Important information
  retained                                                                      represented in a summary

  Older messages    Retained                   Move outside the active window   May be compressed into the
                                                                                summary

  Uses an LLM for   No summarization LLM       No summarization LLM required    Yes
  memory updates    required                                                    

  History size      Grows with conversation    Limited by `k`                   Summary is generally
                                                                                shorter

  Exact message     Retained in the buffer     Only recent messages retained in Not guaranteed; content is
  history                                      the active window                summarized

  Main benefit      Complete history           Recent context control           Condensed context

  Main limitation   Growing history and token  Older details leave the active   Details may be omitted
                    usage                      window                           during summarization
  ---------------------------------------------------------------------------------------------------------

### Easy way to remember

-   **Buffer Memory:** Keep everything.
-   **Window Memory:** Keep only the latest few turns.
-   **Summary Memory:** Keep the important points in a shorter form.

------------------------------------------------------------------------

## 6. Same conversation, three different memory behaviors

Consider this conversation:

1.  Human: My name is Kavya.
2.  AI: Nice to meet you, Kavya.
3.  Human: I work as a Cloud Engineer.
4.  AI: That sounds interesting.
5.  Human: I am learning Kubernetes.
6.  AI: Kubernetes is useful for container orchestration.
7.  Human: I am also learning Terraform.
8.  AI: Terraform helps manage infrastructure as code.

Now compare the three memory types.

### Buffer Memory

Retains all four exchanges:

-   Name: Kavya.
-   Profession: Cloud Engineer.
-   Learning: Kubernetes.
-   Learning: Terraform.

### Window Memory with k = 2

Retains only the latest two exchanges:

-   Learning: Kubernetes.
-   Learning: Terraform.

The earlier name and profession exchanges have moved outside the active
window.

### Summary Memory

May maintain a condensed summary such as:

Kavya is a Cloud Engineer who is learning Kubernetes and Terraform.

The exact summary depends on what the LLM identifies as important.

This example shows the key difference:

-   Buffer retains the full transcript.
-   Window retains a limited recent transcript.
-   Summary retains a condensed representation of the discussion.

------------------------------------------------------------------------

## 7. Which memory should we choose?

  -----------------------------------------------------------------------
  Requirement                         Suitable memory
  ----------------------------------- -----------------------------------
  Need the complete conversation      ConversationBufferMemory
  history                             

  Need only recent conversation turns ConversationBufferWindowMemory

  Need a concise summary of a long    ConversationSummaryMemory
  discussion                          
  -----------------------------------------------------------------------

### Practical scenarios

**Scenario 1: Short customer-support conversation**

A customer asks a few questions about an order. The chatbot needs the
complete discussion to answer follow-up questions.

Suitable choice: **ConversationBufferMemory**

**Scenario 2: A chatbot for quick technical questions**

A user asks several questions, and the latest questions are the most
relevant to the next response.

Suitable choice: **ConversationBufferWindowMemory**

**Scenario 3: A long learning conversation**

A student discusses several topics over a long conversation. The chatbot
needs a concise summary of the important topics and information.

Suitable choice: **ConversationSummaryMemory**

------------------------------------------------------------------------

## 8. Important concepts to remember

### Memory is not the same as permanent storage

These memory concepts describe how conversation context is managed. They
do not automatically guarantee that the conversation will survive an
application restart.

For persistent conversation history, an application needs an appropriate
storage mechanism, such as a database or persistent chat-history store.

### Memory is not the same as model training

Conversation memory does not mean the LLM is retrained after every
message. It manages conversation context that is supplied to the model
during later interactions.

### Memory does not guarantee perfect recall

-   Buffer Memory retains the messages in its history.
-   Window Memory deliberately excludes older turns from its active
    window.
-   Summary Memory can omit details when creating a shorter summary.

The application should choose the memory approach based on how much
historical information it needs.

------------------------------------------------------------------------

## 9. Quick revision for students

  -----------------------------------------------------------------------
  Question                            Answer
  ----------------------------------- -----------------------------------
  Which memory stores the complete    ConversationBufferMemory
  conversation?                       

  Which memory uses `k`?              ConversationBufferWindowMemory

  What does `k=2` mean?               Keep the latest two conversation
                                      turns

  Which memory creates a progressive  ConversationSummaryMemory
  summary?                            

  Which memory can move older turns   ConversationBufferWindowMemory
  outside its active history?         

  Which memory can omit minor details ConversationSummaryMemory
  while summarizing?                  

  Which memory keeps the full message ConversationBufferMemory
  history in its buffer?              
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 10. Final takeaway

The three memory types solve the same broad problem---making previous
conversation information available to an AI application---but they
manage that information differently.

1.  **ConversationBufferMemory** retains the complete conversation.
2.  **ConversationBufferWindowMemory** retains only the latest `k`
    conversation turns.
3.  **ConversationSummaryMemory** uses an LLM to convert the
    conversation into a progressively updated summary.

**Remember this simple formula:**

> Buffer = Everything\
> Window = Recent turns\
> Summary = Important points

------------------------------------------------------------------------

## Note about LangChain versions

LangChain APIs and packages evolve. The notebooks accompanying this
guide use the legacy-style memory classes from `langchain_classic`.
Deprecation warnings and package changes should be monitored, and
current LangChain documentation should be checked before using these
APIs in a new production application.
