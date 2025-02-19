# Replicate Python client

This is a Python client for Replicate. It lets you run models from your Python code or Jupyter notebook, and do various other things on Replicate.

Grab your token from [replicate.com/account](https://replicate.com/account) and authenticate by setting it as an environment variable:

```
export REPLICATE_API_TOKEN=[token]
```

You can run a model and get its output:

```python
$ python
>>> import replicate
>>> model = replicate.models.get("stability-ai/stable-diffusion")
>>> model.predict(prompt="a 19th century portrait of a wombat gentleman")
['https://replicate.com/api/models/stability-ai/stable-diffusion/files/50fcac81-865d-499e-81ac-49de0cb79264/out-0.png']
```

Some models, like [replicate/resnet](/replicate/resnet), receive images as inputs. To pass a file as an input, use a file handle or URL

```python
>>> model = replicate.models.get("replicate/resnet")
>>> model.predict(image=open("mystery.jpg", "rb"))
[['n02123597', 'Siamese_cat', 0.8829364776611328],
 ['n02123394', 'Persian_cat', 0.09810526669025421],
 ['n02123045', 'tabby', 0.005758069921284914]]
```

You can run a model and feed the output into another model:

```python
>>> image = replicate.models.get("afiaka87/laionide-v4").predict(prompt="avocado armchair")
>>> upscaled_image = replicate.models.get("jingyunliang/swinir").predict(image=image)
```

Run a model and get its output while it's running:

```python
model = replicate.models.get("pixray/text2image")
for image in model.predict(prompts="san francisco sunset"):
    display(image)
```

You can start a model and run it in the background:

```python
>>> model = replicate.models.get("kvfrans/clipdraw")
>>> prediction = replicate.predictions.create(
...    version=model.versions.list()[0],
...    input={"prompt":"Watercolor painting of an underwater submarine"})

>>> prediction
Prediction(...)

>>> prediction.status
'starting'

>>> dict(prediction)
{"id": "...", "status": "starting", ...}

>>> prediction.reload()
>>> prediction.status
'processing'

>>> print(prediction.logs)
iteration: 0, render:loss: -0.6171875
iteration: 10, render:loss: -0.92236328125
iteration: 20, render:loss: -1.197265625
iteration: 30, render:loss: -1.3994140625

>>> prediction.wait()

>>> prediction.status
'succeeded'

>>> prediction.output
'https://.../output.png'
```

You can cancel a running prediction:

```python
>>> model = replicate.models.get("kvfrans/clipdraw")
>>> prediction = replicate.predictions.create(
...    version=model.versions.list()[0],
...    input={"prompt":"Watercolor painting of an underwater submarine"})

>>> prediction.status
'starting'

>>> prediction.cancel()

>>> prediction.reload()
>>> prediction.status
'canceled'
```

By default, `model.predict()` uses the latest version. If you want to pin to a particular version, you can get a version with its ID, then call the `predict()` method on that version:

```
>>> model = replicate.models.get("replicate/hello-world")
>>> version = model.versions.get("5c7d5dc6dd8bf75c1acaa8565735e7986bc5b66206b55cca93cb72c9bf15ccaa")
>>> version.predict(text="python")
"hello python"
```

You can list all the predictions you've run:

```
>>> replicate.predictions.list()
[<Prediction: 8b0ba5ab4d85>, <Prediction: 494900564e8c>]
```

## Install

```sh
pip install replicate
```

## Authentication

Set the `REPLICATE_API_TOKEN` environment variable to your API token. For example, run this before running any Python scripts that use the API:

```
export REPLICATE_API_TOKEN=<your token>
```

We recommend not adding it directly to your source code, because you don't want to put your API in source control. If anyone uses your API key, their usage would be charged to your account.

If you have access to the API, [you can find your API key on your dashboard when signed in](https://replicate.com).

## Development

See [CONTRIBUTING.md](CONTRIBUTING.md)
