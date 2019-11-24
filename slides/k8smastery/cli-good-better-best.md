# Kubernetes Management Approaches


- Imperative commands: `run`, `expose`, `scale`, `edit`, `create` deployment
  - Best for dev/learning/personal projects
  - Easy to learn, hardest to manage over time

--

- Imperative objects: `create -f file.yml`, `replace -f file.yml`, `delete`...
  - Good for prod of small environments, single file per command
  - Store your changes in git-based yaml files
  - Hard to automate

--

- Declarative objects: `apply -f file.yml` or `-f dir\`, `diff`
  - Best for prod, easier to automate
  - Harder to understand and predict changes


