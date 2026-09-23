## Important Points
### 15.1 Challenges Encountered During Development (original findings)

| Challenge | Root Cause | Resolution |
|---|---|---|
| API Gateway returned path not found | Path matching too strict (endswith) | Relaxed matching + printed full event for debugging |
| POST /scan/trigger returned 500 | Missing lambda:InvokeFunction permission | Added inline policy + environment variable for function name |
| EventBridge produced no CloudWatch Logs | Missing resource-based permission on Lambda | Added events.amazonaws.com permission with SourceArn |
| Dashboard showed only mocks | USE_MOCKS still true + wrong base URL | Updated .env and endpoints.js flag |
| IAM role too broad initially | Speed of development | Replaced with scoped custom policy before finalization |
