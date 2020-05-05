# understand-github-actions-on-paths

Example repository to understand how does list of changed files is generated in GitHub Actions. Also investigates Travis CI's `TRAVIS_COMMIT_RANGE` and CircleCI's `pipeline.git.revision` and `pipeline.git.base_revision`.

Commit graph:

![Commit graph](https://github.com/orangain/understand-github-actions-on-paths/raw/master/commit-graph.svg)

## GitHub Actions

Each commit creates a new file and add a new workflow whose trigger of `on.push.paths` and `on.pull_request.paths` is a filter which exactly match the newly created file. For example, [Add A](https://github.com/orangain/understand-github-actions-on-paths/commit/fadaa497c962056a88b44bd615cdd059f50b5e76) creates a file `A` and add a workflow which will be triggered when the file `A` is changed. This enables us to know which commits are considered to be changed by GitHub Actions.


| Action | Workflows triggered |
|--------|--------|
| 1️⃣ Push A and B to master branch | `push`: A and B |
| 2️⃣ Push C and D to master branch | `push`: C and D |
| 3️⃣ Force push D2 and E to master branch | `push`: D2 and E (Read [this question](https://github.community/t5/GitHub-Actions/Workflow-paths-on-forced-pushs/td-p/49576) about force push) |
| 4️⃣ Push F and G to feature branch | `push`: F and G |
| 5️⃣ Create pull request from feature branch to master | `pull_request`: F and G |
| 6️⃣ Push H and I to feature branch | `push`: H and I <br>`pull_request`: F, G, H and I |
| 7️⃣ Merge feature branch into master | `push`: F, G, H and I |

## Travis CI [![Build Status](https://travis-ci.org/orangain/understand-github-actions-on-paths.svg?branch=master)](https://travis-ci.org/orangain/understand-github-actions-on-paths)

Travis CI provides environment variable `TRAVIS_COMMIT_RANGE` that help us determine which files are changed.
See: https://docs.travis-ci.com/user/environment-variables/

| Action | `TRAVIS_COMMIT_RANGE` |
|--------|--------|
| 1️⃣ Push A and B to master branch | `push`: `d4059e8e2cea...6d0c4f1214c2` |
| 2️⃣ Push C and D to master branch | `push`: `6d0c4f1214c2...4157b336df53` |
| 3️⃣ Force push D2 and E to master branch | `push`: `4157b336df53...ddabd4e73b5b` |
| 4️⃣ Push F and G to feature branch | `push`: `055cc3b8aa24^...5f7e74638d26` |
| 5️⃣ Create pull request from feature branch to master | `pull_request`: `ddabd4e73b5b3de9e5e3a28bad60c42a5d9a3027...5f7e74638d26a9ad108478cae13541b58392c3e2` |
| 6️⃣ Push H and I to feature branch | `push`: `5f7e74638d26...85ed9973b259` <br> `pull_request`: `ddabd4e73b5b3de9e5e3a28bad60c42a5d9a3027...85ed9973b25967a78ac9a9af0ac159123e0fb23c` |
| 7️⃣ Merge feature branch into master | `push`: `ddabd4e73b5b...d4f5360c10b1` |

## CircleCI [![CircleCI](https://circleci.com/gh/orangain/understand-github-actions-on-paths.svg?style=svg)](https://circleci.com/gh/orangain/understand-github-actions-on-paths)

CircleCI provides pipeline variables `pipeline.git.revision` and `pipeline.git.base_revision` that help us determine which files are changed.
See: https://circleci.com/docs/2.0/pipeline-variables/

| Action | `pipeline.git.revision` | `pipeline.git.base_revision`
|--------|--------|--------|
| 1️⃣ Push A and B to master branch | `6d0c4f1214c24dd99e3f722a3623dcc5c489b359` | `d4059e8e2cea02d8213ce6c47db05b73d343914d` |
| 2️⃣ Push C and D to master branch | `4157b336df5320914d142fe09b42f5dab3229dcb` | `6d0c4f1214c24dd99e3f722a3623dcc5c489b359` |
| 3️⃣ Force push D2 and E to master branch | `ddabd4e73b5b3de9e5e3a28bad60c42a5d9a3027` | `4157b336df5320914d142fe09b42f5dab3229dcb` |
| 4️⃣ Push F and G to feature branch | `5f7e74638d26a9ad108478cae13541b58392c3e2` | (Empty) |
| 5️⃣ Create pull request from feature branch to master | (Build not triggered) | (Build not triggered) |
| 6️⃣ Push H and I to feature branch | `85ed9973b25967a78ac9a9af0ac159123e0fb23c` | `5f7e74638d26a9ad108478cae13541b58392c3e2` |
| 7️⃣ Merge feature branch into master | `d4f5360c10b179b95f734f0b82348f72f7699c10` | `ddabd4e73b5b3de9e5e3a28bad60c42a5d9a3027` |

