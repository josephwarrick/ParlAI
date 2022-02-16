### Initial thoughts:
I've never used bitbucket pipelines before, but I expect it should be relatively similar to other CI/CD systems I've used.
I'm not familiar with ParlAI at all.
From speaking with Joseph Hamala I recall that most of Helm's code is in Typescript, so I'm mildly surprised to see a python project, although I suppose the laguange doesn't matter a huge amount for the CI/CD pipeline.
I have some experience with Python, but mostly smaller projects, nothing as large as ParlAI appears to be.
I see that there is a `.circleci/config.yml`; this should give me some idea of what the project is expected to do, and a model to base the bitbucket CI pipeline off of.

### First attempts at running locally:
- Running the tests with `pytest` fails with an unexpected `No module named 'transformers'` error. If I `pip install transformers` then the tests seem to run. I'm not sure why the `transformers` package is not listed in `requirements.txt`. Since I don't know, I'll leave it out for now.
- The complete set of tests seems to be pretty slow. I stop the initial run after ~30 minutes, the tests report that they are ~38% completed. I see they're CPU bound, on a single process, perhaps there's an opportunity to run tests in parallel and speed them up?
- I see there's a subset of tests that are marked as unit tests. The existing workflow on the "real" repo seems to be that unit tests are always run on pull requests, and other tests are only run when relevant code changes?
- Hmm... I'm getting a `KeyError: 'rouge_L'` error on two of the `test_eval_model` tests. I haven't changed any code, so I would expect all of the tests to pass.
- I see a `python -c "import nltk; nltk.download('punkt')"` command in the setup of the existing circle ci config. Running this and _then_ running the unit tests, all the unit tests pass (in ~14 minutes).
