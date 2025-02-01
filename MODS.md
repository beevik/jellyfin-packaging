Steps to modify jellyfin
========================

Use these steps to build a docker image of jellyfin that runs as a non-root
user named `jellyfin` with a custom userid and groupid (503). This is useful
if you're running jellyfin against an NFS-mounted volume where the content
files are owned by user with a specific user-id and group-id.

1. Fork `jellyfin/jellyfin-packaging` into `beevik/jellyfin-packaging`.

2. Clone the forked repo.

```
git clone git@github.com:beevik/jellyfin-packaging.git
```

3. Check out the latest stable branch (`v10.10.5` at the time of writing).

```
git co -b v10.10.5 v10.10.5-202410261333
```

4. Get git submodules.

```
git submodule update --init
```

5. Make sure submodules match the main repo.

```
./checkout.py v10.10.5
```

6. Create a new branch.

```
git co -b brett/jellyfin
```

7. Modify `docker/Dockerfile` by adding the following just above the
   `EXPOSE 8096` line:

```
# Create jellyfin user and group
RUN groupadd -g 503 jellyfin \
 && useradd -g 503 -u 503 -M jellyfin \
 && chown -R jellyfin:jellyfin /jellyfin ${JELLYFIN_DATA_DIR} ${JELLYFIN_CACHE_DIR}

# Don't run jellyfin as root
USER jellyfin:jellyfin
```

This assumes a user and group id of 503 for the jellyfin account. Modify
as desired.

8. Build the docker image.

```
./build.py brett docker amd64 --local
```

9. Tag the resulting image.

```
docker tag jellyfin/jellyfin:brett-amd64 brett/jellyfin
```

10. Use my custom `docker-compose` configuration to launch jellyfish with
an nginx reverse HTTPS proxy.
