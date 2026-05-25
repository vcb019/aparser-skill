# A-Parser HTTP API Reference

## Overview

A-Parser provides an HTTP API for external control and integration.

**Endpoint:** `POST http://IP:9091/API`  
**Content-Type:** `application/json`  
**Required headers:** `content-length`, `content-type: application/json`

### Request format
```json
{
  "password": "your_password",
  "action": "methodName",
  "data": { }
}
```

### Response format
```json
{"success": 1, "data": "..."}
```
`success`: 1 = OK, 0 = error.

---

## API Methods

### ping
Health check.
```json
{ "action": "ping" }
```
Response: `{"success": 1, "data": "pong"}`

---

### oneRequest
Single parsing request.
```json
{
  "action": "oneRequest",
  "data": {
    "parser": "SE::Google",
    "preset": "default",
    "query": "keyword",
    "options": {
      "pagecount": 2,
      "useproxy": 1
    }
  }
}
```
Returns parsed results for one query.

---

### bulkRequest
Mass parsing with multiple queries.
```json
{
  "action": "bulkRequest",
  "data": {
    "parser": "SE::Google",
    "preset": "default",
    "queries": ["keyword1", "keyword2", "keyword3"],
    "threads": 3
  }
}
```
Returns array of results, one per query.

---

### addTask
Add task to queue. Returns task UID.
```json
{
  "password": "pass",
  "action": "addTask",
  "data": {
    "preset": "default",
    "configPreset": "default",
    "parsers": [
      ["SE::Google", "default",
        {"type": "override", "id": "pagecount", "value": 1},
        {"type": "override", "id": "useproxy", "value": true}
      ]
    ],
    "resultsFormat": "$p1.serp.format('$link;$anchor\\n')",
    "resultsSaveTo": "file",
    "resultsFileName": "$datefile.format().csv",
    "resultsUnique": "string",
    "queriesFrom": "text",
    "queries": ["keyword1", "keyword2"],
    "queryFormat": ["$query"],
    "uniqueQueries": true,
    "doLog": "db",
    "moreOptions": true,
    "resultsPrepend": "Link;Anchor\n",
    "resultsAppend": "",
    "queryBuilders": [],
    "resultsBuilders": [],
    "configOverrides": [],
    "removeOnComplete": false,
    "removeOnRestart": false,
    "prio": 5
  }
}
```

**queriesFrom:** `"text"` (inline queries array) or `"file"` + `"queriesFile": ["queries/file.txt"]`.

**To run a saved preset** — just specify `preset` name and `queries`:
```json
{"preset": "My Preset", "queriesFrom": "text", "queries": ["google.com"]}
```

Returns: `{"success": 1, "data": "697403"}` — task UID as string.

---

### info
System status.
```json
{ "action": "info" }
```
Returns: version, task queue, active threads, available parsers list.

---

### getParserPreset
Get parser configuration.
```json
{
  "action": "getParserPreset",
  "data": {
    "parser": "SE::Google",
    "preset": "default"
  }
}
```
Returns all parser settings for the given preset.

---

### getProxies
List active proxies.
```json
{ "action": "getProxies" }
```
Returns array of active proxies with their types from all checkers.

---

### getTaskState
Get task status and statistics.
```json
{
  "password": "pass",
  "action": "getTaskState",
  "data": { "taskUid": "697403" }
}
```
For multiple tasks: `"taskUid": ["22", "23", "31"]`

Response:
```json
{
  "success": 1,
  "data": {
    "status": "completed",
    "state": {
      "queriesDoneCount": 254,
      "queriesCount": 254,
      "totalFail": 2,
      "resultsCount": 31079,
      "uniqueResultsCount": 656,
      "avgSpeed": 802,
      "curSpeed": 846,
      "activeThreads": 0,
      "runTime": 19
    }
  }
}
```
**status values:** `waitSlot`, `work`, `paused`, `stopped`, `completed`

---

### getTaskConf
Get full task configuration.
```json
{
  "password": "pass",
  "action": "getTaskConf",
  "data": { "taskUid": "697403" }
}
```
Returns: parsers, queries, resultsFileName (resolved), all settings.

---

### getTaskResultsFile
Get single-use download URL for results.
```json
{
  "password": "pass",
  "action": "getTaskResultsFile",
  "data": { "taskUid": "697403" }
}
```
Response: `{"success": 1, "data": "http://127.0.0.1:9091/downloadResults?fileName=...&token=..."}`

Note: Works only with static filenames or `$datefile.format()`. Use `isStaticTemplate()` in filename for dynamic names.

---

### getTasksList
List task UIDs.
```json
{
  "password": "pass",
  "action": "getTasksList",
  "data": { "completed": "1" }
}
```
Pass `"completed": "1"` for finished tasks, omit for active tasks.

---

### getParserInfo
Get available result fields for a parser.
```json
{
  "password": "pass",
  "action": "getParserInfo",
  "data": { "parser": "SE::Google" }
}
```
Returns flat fields and array fields with sub-columns.

---

### getAccountsCount
Get count of active Yandex accounts.
```json
{ "password": "pass", "action": "getAccountsCount" }
```
Response: `{"success": 1, "data": {"SE::Yandex": 18}}`

---

### changeTaskStatus
Control task execution.
```json
{
  "password": "pass",
  "action": "changeTaskStatus",
  "data": {
    "taskUid": "697403",
    "toStatus": "stopping"
  }
}
```
**toStatus values:** `"starting"`, `"pausing"`, `"stopping"`, `"deleting"`

---

### changeProxyCheckerState
Enable or disable a proxy checker.
```json
{
  "password": "pass",
  "action": "changeProxyCheckerState",
  "data": { "checker": "default", "state": 1 }
}
```
`state`: 1 = enabled, 0 = disabled.

---

### deleteTaskResultsFile
Delete result file for a task.
```json
{
  "password": "pass",
  "action": "deleteTaskResultsFile",
  "data": { "taskUid": "697403" }
}
```

---

### moveTask
Reposition task in queue.
```json
{
  "password": "pass",
  "action": "moveTask",
  "data": { "taskUid": "697403", "direction": "start" }
}
```
**direction values:** `"start"`, `"end"`, `"up"`, `"down"`

---

### update
Update A-Parser to latest version. A-Parser restarts automatically.
```json
{ "password": "pass", "action": "update" }
```

---

## Client Libraries

### Node.js client
```bash
npm install a-parser-client
```
```javascript
const AParserClient = require('a-parser-client');
const AParser = new AParserClient('http://127.0.0.1:9091/API', 'password');

AParser.ping().then(reply => console.log(reply.data));

(async () => {
  const taskUid = (await AParser.addTask({ ... })).data;
  const state = await AParser.getTaskState({ taskUid });
})();
```
All API methods are implemented. Requires Node.js ≥ 10.x.

### PHP client
```bash
wget https://github.com/a-parser/api-php/raw/master/aparser-api-php-client.php
```
```php
require_once 'aparser-api-php-client.php';
$aparser = new Aparser('http://127.0.0.1:9091/API', 'pass');
$aparser->ping();
$aparser->oneRequest('keyword', 'SE::Google', 'default');
$taskUid = $aparser->addTask('default', false, 'text', ['keyword1', 'keyword2']);
$aparser->getTaskState($taskUid);
$aparser->changeTaskStatus($taskUid, 'deleting');
```
Alternative: `composer require reset-button/a-parser-api-php-client`. Requires PHP ≥ 5.3.

### Python client
```bash
pip install a-parser
```
```python
from a_parser import AParser
aparser = AParser('http://127.0.0.1:9091/API', 'password')

print(aparser.ping())  # {'success': 1, 'data': 'pong'}

task_uid = aparser.addTask(
    [['SE::Google', 'default',
      {'type': 'override', 'id': 'pagecount', 'value': 1}]],
    'default', 'text', 'keyword',
    resultsFormat='$p1.serp.format("$link\\n")'
)['data']

aparser.waitForTask(task_uid)  # blocks until completed (polls every 5s)
print(aparser.getTaskResultsFile(task_uid))
```
`waitForTask(uid, interval=5)` — polls until status is `completed`. Requires Python 2.7 or ≥ 3.8.

---

## Node.js Raw Example (no client library)

```javascript
const http = require('http');

function apiRequest(action, data = {}) {
  return new Promise((resolve, reject) => {
    const body = JSON.stringify({ password: 'your_password', action, data });
    const options = {
      host: '127.0.0.1', port: 9091, path: '/API', method: 'POST',
      headers: { 'content-type': 'application/json', 'content-length': Buffer.byteLength(body) }
    };
    const req = http.request(options, res => {
      let result = '';
      res.on('data', chunk => result += chunk);
      res.on('end', () => resolve(JSON.parse(result)));
    });
    req.on('error', reject);
    req.write(body);
    req.end();
  });
}

(async () => {
  // Add task:
  const addRes = await apiRequest('addTask', {
    preset: 'default',
    configPreset: 'default',
    parsers: [['SE::Google', 'default']],
    resultsFormat: '$query\t$p1.totalcount\n',
    resultsSaveTo: 'file',
    resultsFileName: '$datefile.format().txt',
    queriesFrom: 'text',
    queries: ['buy laptop', 'cheap laptop'],
  });
  const taskUid = addRes.data;

  // Wait for completion:
  while (true) {
    const stateRes = await apiRequest('getTaskState', { taskUid });
    if (stateRes.data.status === 'completed') break;
    await new Promise(r => setTimeout(r, 2000));
  }

  // Get download link:
  const fileRes = await apiRequest('getTaskResultsFile', { taskUid });
  console.log('Download:', fileRes.data);
})();
```

---

## Redis API

Alternative high-performance API using Redis as message queue.

**Use case:** When you need `oneRequest`/`bulkRequest` at scale with async processing, multiple A-Parser instances, or timeout support.

**Setup:**
1. Install and run Redis
2. Create preset for `API::Server::Redis` parser — set Redis Host, Port, Queue Key (`aparser_redis_api`)
3. Add task with `API::Server::Redis`, queries: `{num:1:N}` where N = thread count

**Submit request:**
```
lpush aparser_redis_api '["unique_id", "Net::HTTP", "default", "https://example.com"]'
```
Request format: `[queryId, parser, preset, query, overrideOpts?, apiOpts?]`

**Get result (blocking):**
```
blpop aparser_redis_api:unique_id 0
```

**Get result (async):**
```
lpop aparser_redis_api:unique_id
```
Returns `nil` if not yet processed.

**Output to shared queue** (for single-threaded result processing):
```
lpush aparser_redis_api '["uid1", "Net::HTTP", "default", "https://ya.ru", {}, {"output_queue": "aparser_results"}]'
blpop aparser_results 0
```

**docker-compose with Redis:**
```yaml
version: '3'
services:
  a-parser:
    image: aparser/runtime:latest
    command: ./aparser
    volumes: [./aparser:/app]
    ports: ["9091:9091"]
  redis:
    image: redis:latest
    command: redis-server --requirepass YOUR_REDIS_PASSWORD
    ports: ["6379:6379"]
```
Use service name `redis` as Redis Host inside Docker network.
