# Vectors In Pinecone
This was just an excuse for me to use the Pinecone console.

My initial impression can be summed up as WOW. Serverless takes away my operational woes and the ease at which the console allows you to ingest and query/sample data is impressive. I'm used to having a web UI that breaks or VI instances in EC2 that need help so this is a refreshing experience.

I had never messed with a vector database, but Pinecone's website/docs and console made it easy to understand and get to work. Zero to hero fast in my mind. Ha!

<br/>

# Serverless
So yeah, let me repeat that, serverless is sweet. It's very developer-friendly. In under a few minutes, I went from having nothing to having a live queryable index of sample data.

![image](https://github.com/scottsappen/VectorsInPinecone/assets/2436969/65be827f-f69e-4c9c-9afb-510f1de66a5c)

<br/>

# Sample Data
Instead of creating my own index, which I'll get to, I decided to use the sample data button. It looks like a pretty cool data set and gave me a chance to use the text to vector OpenAI embedding model too.

I did some digging to learn about the embeddings model [text-embedding-ada-002](https://openai.com/blog/new-and-improved-embedding-model). I'll get back to that later with the OpenAI API.

![image](https://github.com/scottsappen/VectorsInPinecone/assets/2436969/a2049a86-4cf4-4497-9577-5cd42dad6f31)

<br/>

# Index
I absolutely love the browser interface for the indexed data.

<img width="1416" alt="image" src="https://github.com/scottsappen/VectorsInPinecone/assets/2436969/f3d3f600-3982-4db9-8780-91c59aeeea58">

Let's get on with querying the data now.

<br/>

# Enter OpenAI
Well, the model used to create the embeddings is an OpenAI text to vector model. That's given to us [text-embedding-ada-002](https://openai.com/blog/new-and-improved-embedding-model). In order to query my Pinecone index, I'll need to convert my query into a vector. Using the same embedding model will help or I'd definitely get some surprises! :)

I found a quick and dirty way to do it using Open AI's API. I'll just need to setup an API key and get rolling.

<br/>

# OpenAI embedding call
I decided to ask my index data for Harry Pottery movies, for lack of a better imagination. I do love those movies. So I created a shell script to make the call to OpenAI.

My query I'm going to vectorize: "What movies contain Harry Potter".

Notice I use the same embedding model text-embedding-ada-002 that Pinecone gave me in the sample data.

```shell
scottsm1home@scotts-mbp OpenAI % cat openai_encode_harrypotter.sh

curl https://api.openai.com/v1/embeddings \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY_HERE" \
  -d '{
    "input": "What movies contain Harry Potter",
    "model": "text-embedding-ada-002"
  }'
```

The output is straightforward, vectorized query data.
```shell
{
  "object": "list",
  "data": [
    {
      "object": "embedding",
      "index": 0,
      "embedding": [
        -0.0102287065,
        -0.031506605,
        -0.014057634,
        .............
        -0.012491876,
        -0.011021842,
        -0.008888582
      ]
    }
  ],
  "model": "text-embedding-ada-002-v2",
  "usage": {
    "prompt_tokens": 5,
    "total_tokens": 5
  }
}
```

I'll pipe this to an output file for convenience.

```shell
scottsm1home@scotts-mbp OpenAI % ./openai_encode_harrypotter.sh >> openai_encodedvector.json
```

Come to think of it, I'm really only interested in the embeddings data so I created this python script to get that for me. I removed the starting and end braces so I can copy/paste this into Pinecone's query interface.

```shell
scottsm1home@scotts-mbp OpenAI % cat extractvector_harrypotter.py
```

```python
import json

# Step 1: Read the JSON file
with open('./openai_encodedvector.json', 'r') as file:
    data = json.load(file)

# Step 2: Extract the embedding
embedding = data['data'][0]['embedding']

# Step 3: Convert the embedding list to a string and remove the brackets
embedding_str = str(embedding)[1:-1]

# Step 4: Write the embedding to a new file
with open('./embeddedfieldonly_harrypotter.json', 'w') as new_file:
    new_file.write(embedding_str)
```

That new file looks good and has the query vector I want.
```shell
scottsm1home@scotts-mbp OpenAI % cat embeddedfieldonly_harrypotter.json
-0.0102287065, -0.031506605, -0.014057634, -0.050213654, -0.020744583, 0.002644353, -0.009893675, -0.04247375, -0.02090868, -0.00074655545, 0.024970079, 0.012095309, 0.010686811, 0.0046152254, 0.007466409, 0.0036067131, 0.028306715, -0.0022358203, 0.021852238, -0.036402.........
```

<br/>

# Query the Pinecone Index
Let's grab that data to our clipboard

```shell
scottsm1home@scotts-mbp OpenAI % cat embeddedfieldonly_harrypotter.json | pbcopy
```

And voila, I query the top K 8 rows (I think there are 8 movies) and it brings me back 7 Harry Potter films. Maybe there are only 7 or maybe the data set only contained 7. I'll have to investigate which one is missing.

<img width="1415" alt="image" src="https://github.com/scottsappen/VectorsInPinecone/assets/2436969/6e2adde6-60de-4fa9-b037-5e3fa8054e13">

<br/>

It appears I got:<br/>
title: "Harry Potter and the Deathly Hallows: Part 2"<br/>
title: "Harry Potter and the Goblet of Fire"<br/>
title: "Harry Potter and the Order of the Phoenix"<br/>
title: "Harry Potter and the Half-Blood Prince"<br/>
title: "Harry Potter and the Chamber of Secrets"<br/>
title: "Harry Potter and the Deathly Hallows: Part 1"<br/>
title: "Harry Potter and the Sorcerer's Stone"

So anyway this is an awesome starta and I'm going to get cranking on more. I can see the value in this, can you imagine if your vectorized dataset was a list of consumer products or medicines or anything else that could be matched with stale LLM models for up-to-date chat AI bots? The use-cases are endless!

<br/>
