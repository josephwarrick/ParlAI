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

### What is in the CI process
- It's not the most impressive CI process. I tried several other things that I think would be better, but they kept not working on bitbucket, so this is a working solution without spending too much time on the problem.
- This is a very basic CI process that runs unit tests every time new code is added to the `main` branch, as well as running the unit tests on every pull request.
- The original repo tests the code against several different versions of torch, CPU, GPU, etc. This setup does not perform the same range of tests. In some cases this is not possible. For example, bitbucket doesn't have any GPU specific infrastructure that we can test on, while their "real" test infrastructure does. The most probable real world use case for something like this would probably be that we extended the code to fit a very specific use case, and in that case we can tailor our tests to this same very specifc use case, instead of needing a broadly applicable solution like the existing code base does.
- The "real" repo uses codecov to track the coverage of the code. I've left that out, since I didn't want to set up and link a codecov account.
- The "real" code base deploys a website, and also runs tests against that website. This process does not include that.

### Issues with setting up bitbucket pipelines:
Perhaps I did not spend enough time learning bitbucket, but these are some of the thigs that I expected to work on bitbucket, but I couldn't get them to work in a reasonable amount of time.
- bitbucket doesn't support absolute paths in cache directories and artifact directories
I first attempted to use the home directory for a couple things, like the virtual environment location and the natural language toolkit download, but bitbucket doesn't allow absolute paths to cache directories and artifacts, only relative paths. This means that I need to put these things in the local directory.
- bitbucket has a maximum limit of 1GB for artifact size
I initially tried to split the build process up into multiple steps: one to install dependencies, one to actually run the tests, etc which would simplify the configuration. For example, there could be a common install step, and then the tests could be run in parallel across several different versions of pytorch that we need to support.

However, the only built in way to share state between steps is to define them as artifacts. There is a limit of 1Gb on the size of artifacts, so this won't work in this case.

That is, after installing the necessary packages, the directory containing the packages is larger than 1Gb, so it can't be shared with the next step as an artifact. If we wanted to keep going down this path we would probably need to build our own caching system by zipping the directory, uploading it to S3, then downloading and unzipping in the next step. But that's too much effort for this demo.

We could build the entire build/install/test process around docker images (that might be my first preference anyway), but the setup of the propmt seemed to imply that you wanted CI set up sepearately from a docker build.
- bitbucket only allows 50 minutes of build time per month, until you start paying $15 or more per month. This seems really low compared to i.e. github, which allows 2,000 minutes of build time per month before needing to pay.
