# Multi-architecture build examples

Two tiny projects that implement the **same mini-pastebin HTTP API** (see
[`docs/API.md`](docs/API.md)). They are used by the three multi-architecture
build examples described in the [root README](../README.md).

| Project                                                 | Language / framework       | Build style  | Container port | Host port (compose) | Swagger UI |
| ------------------------------------------------------- | -------------------------- | ------------ | -------------- | ------------------- | ---------- |
| [`python-fastapi-multiarch/`](python-fastapi-multiarch) | Python 3.14 / FastAPI      | single stage | 8080           | 8001                | `/docs`    |
| [`csharp-aspnet/`](csharp-aspnet)                       | C# / .NET 8 (ASP.NET Core) | multi-stage  | 8080           | 8201                | `/swagger` |

The C# image is a multi-stage build: **the final image contains only the
published artifact** (plus its runtime libraries). The Python image is a single
stage build: **the final image contains the source code of the project**.

Every implementation stores data **in memory only** — there is no database and
no external service. Restarting a container clears all pastes.

---

## How the examples map to the workflows

| Workflow                                                                        | Source used                 | Image platforms                                | Published tag             |
| ------------------------------------------------------------------------------- | --------------------------- | ---------------------------------------------- | ------------------------- |
| [`python-fastapi-multiarch`](../.github/workflows/python-fastapi-multiarch.yml) | `python-fastapi-multiarch/` | `linux/arm64` only                             | `python-arm64`            |
| [`csharp-aspnet`](../.github/workflows/csharp-aspnet.yml)                       | `csharp-aspnet/`            | `linux/amd64` + `linux/arm64` (native, merged) | `csharp-multiarch-native` |
| [`pastebin-csharp-qemu`](../.github/workflows/pastebin-csharp-qemu.yml)         | `csharp-aspnet/`            | `linux/amd64` + `linux/arm64` (QEMU)           | `csharp-multiarch-qemu`   |

The two C# examples intentionally build the **same** `csharp-aspnet/` source so
you can compare the native and QEMU build strategies directly.

All tags are published to the Docker Hub repository configured through the
`DOCKER_IMAGE` variable, for example
`darktohka/multiarch-lab:python-arm64`.

---

## Running an example locally

Compose builds for your host architecture only. It is meant for iterating on the
app, not for producing a multi-arch image — use `docker buildx` for that (see
the [root README](../README.md)).

```bash
# Python app on http://localhost:8001
DOCKER_IMAGE=your-dockerhub-username/multiarch-lab \
  docker compose -f python-fastapi-multiarch/docker-compose.yml up --build -d

# C# app on http://localhost:8201
DOCKER_IMAGE=your-dockerhub-username/multiarch-lab \
  docker compose -f csharp-aspnet/docker-compose.yml up --build -d
```

Then talk to it:

```bash
curl -s -X POST http://localhost:8001/paste \
  -H 'Content-Type: application/json' \
  -d '{"url":"https://www.docker.com/"}'
# {"code":"jZTnNZmo","short_url":"/jZTnNZmo","long_url":"https://www.docker.com/"}

curl -i http://localhost:8001/jZTnNZmo
# HTTP/1.1 302 Found
# location: https://www.docker.com/
```

See [`docs/API.md`](docs/API.md) for the full API contract.
