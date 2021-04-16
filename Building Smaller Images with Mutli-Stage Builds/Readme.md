![image](https://user-images.githubusercontent.com/44756128/115048522-43d5b600-9e9f-11eb-92fe-919235600496.png)

# Introduction
Containers are made of layers. Compile and install operations performed in the image, add to the layers, increasing the size of the container.

Instead of keeping all of those layers into the final image, you can split those steps off, and only use the finished product.

Docker provides this capability through multi-stage builds. We will build an image the usual way, and inspect the image to see how it is put together. We'll then convert the Dockerfile to use Multi-Stage builds, and see how the new image compares.

Log in to the server using ssh:
```sh
ssh username@<PUBLIC_IP_ADDRESS>
```

# Do Prep Work in the Image
Change to the notes directory:
```sh
cd notes
```

Check the Dockerfile:
```sh
cat Dockerfile
```

Build an image using the file:
```sh
docker build -t notesapp:default .
```

![image](https://user-images.githubusercontent.com/44756128/115049374-45ec4480-9ea0-11eb-8e71-0c24219f9cc5.png)

Set a variable to view the layers of the image:
```sh
export showlayers='{{ range .RootFS.Layers }}{{ printLn }}{{end}}'
```

Set a variable to show the size of the image:
```sh
export showSize='{{ .Size }}'
```

Show the image layers:
```sh
docker inspect -f "$showLayers" notesapp:default
```

Count the number of layers:
```sh
docker inspect -f "$showLayers" notesapp:default | wc -l
```

![image](https://user-images.githubusercontent.com/44756128/115049506-6c11e480-9ea0-11eb-83f8-de1ccd8ea797.png)

OUTPUT:
```
[
    {
        "Id": "sha256:ddce90dc653c714465400e8d20495dcb8e86efc960623851d7c8b114db47fc81",
        "RepoTags": [
            "notesapp:default"
        ],
        "RepoDigests": [],
        "Parent": "sha256:3ba40ae597208151879a50610cfdc63b2327317344d731b21c7d48ee5050fcd5",
        "Comment": "",
        "Created": "2021-04-16T15:40:46.633239506Z",
        "Container": "f52a4ce7e914fd7c0509789cc43aa9aec90b4e48678bba99c797e834e22ba910",
        "ContainerConfig": {
            "Hostname": "f52a4ce7e914",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/pybase/bin:/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "GPG_KEY=E3FF2839C048B25C084DEBE9B26995E310250568",
                "PYTHON_VERSION=3.9.4",
                "PYTHON_PIP_VERSION=21.0.1",
                "PYTHON_GET_PIP_URL=https://github.com/pypa/get-pip/raw/29f37dbe6b3842ccd52d61816a3044173962ebeb/public/get-pip.py",
                "PYTHON_GET_PIP_SHA256=e03eb8a33d3b441ff484c56a436ff10680479d4bd14e59268e67977ed40904de",
                "PYBASE=/pybase",
                "PYTHONUSERBASE=/pybase"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"flask\" \"run\" \"--port=80\" \"--host=0.0.0.0\"]"
            ],
            "Image": "sha256:3ba40ae597208151879a50610cfdc63b2327317344d731b21c7d48ee5050fcd5",
            "Volumes": null,
            "WorkingDir": "/app/notes",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "DockerVersion": "20.10.6",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/pybase/bin:/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "GPG_KEY=E3FF2839C048B25C084DEBE9B26995E310250568",
                "PYTHON_VERSION=3.9.4",
                "PYTHON_PIP_VERSION=21.0.1",
                "PYTHON_GET_PIP_URL=https://github.com/pypa/get-pip/raw/29f37dbe6b3842ccd52d61816a3044173962ebeb/public/get-pip.py",
                "PYTHON_GET_PIP_SHA256=e03eb8a33d3b441ff484c56a436ff10680479d4bd14e59268e67977ed40904de",
                "PYBASE=/pybase",
                "PYTHONUSERBASE=/pybase"
            ],
            "Cmd": [
                "flask",
                "run",
                "--port=80",
                "--host=0.0.0.0"
            ],
            "Image": "sha256:3ba40ae597208151879a50610cfdc63b2327317344d731b21c7d48ee5050fcd5",
            "Volumes": null,
            "WorkingDir": "/app/notes",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 1000835744,
        "VirtualSize": 1000835744,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/0ffdd6122792270ef82c2bd7dd2f860723acc2c588b3bce6cebbf5d8c3c97923/diff:/var/lib/docker/overlay2/112b728fb69581348ef3bcff14f65abf25f9389a1644bbda643dd00344c8a41a/diff:/var/lib/docker/overlay2/ab5eb9fa25beb92e9ddf6d0ee94a3cff5c76cf75d7fae3426da7ce4495e2032b/diff:/var/lib/docker/overlay2/c2da5ea5a22978abf0781bd51dd84359c433b10f6bb75e25b63c0c7db5dddcfc/diff:/var/lib/docker/overlay2/c628d7a803b53c3c39a7003f9a16d1c9991968ff549140d7b8b65e25478462d5/diff:/var/lib/docker/overlay2/74fd90e30649aa1637ce4dc72986a772a5c40243f6c2d55937e68d4be9861f43/diff:/var/lib/docker/overlay2/f291bcae0b6035dca7683f1a5681fd7d70712ccb927e16438ef778bd0eb1818b/diff:/var/lib/docker/overlay2/c2b8df05cc93265f24fdcd683ceced81a1fce6a148bf9171a00b63d4b6bdf53c/diff:/var/lib/docker/overlay2/d8188d01f0512d08629b07446c873b7a2cdd0cd14f5c861e5253021954cd0db9/diff:/var/lib/docker/overlay2/aa6e45b45adba9e6b7c38b414eb4cf2afb576ff924a74a3abcbf063b5ed0ec34/diff:/var/lib/docker/overlay2/c069509f7873adbfe1a1d7323d977a3759555da880e9e1c44cb3f903d2fe5450/diff:/var/lib/docker/overlay2/7a136a8cbad45f23aa9d905fd9aa728b107bc359f0db8ae71ad64f5fe474da9c/diff:/var/lib/docker/overlay2/7da39d84ba6577289d5af555b29ff5500dcde7c2ac9f6817cab98801a21f3211/diff",
                "MergedDir": "/var/lib/docker/overlay2/f4671d3c337bd7b3690180b4619af3a31d98167dc43f4d582c938c1906fb11ef/merged",
                "UpperDir": "/var/lib/docker/overlay2/f4671d3c337bd7b3690180b4619af3a31d98167dc43f4d582c938c1906fb11ef/diff",
                "WorkDir": "/var/lib/docker/overlay2/f4671d3c337bd7b3690180b4619af3a31d98167dc43f4d582c938c1906fb11ef/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:e2c6ff46235709f5178ab7c1939f4fba7237ffde84e13b1582fa5f0837c1d978",
                "sha256:26270c5e25fa4569f37428604f708f8863a171ec2499a066241a641017971308",
                "sha256:a42439ce96509df457152574716e3c07a8d5640fe9c16f5d4fb8854f72ce357a",
                "sha256:5d5962699bd5f016d81fc6d623cb0bc1f3853efdc48b6958053122caffc425de",
                "sha256:651326e9f1caf67559db870e293b37ca6f965da6845765b6168e86b665b2e363",
                "sha256:8d18b38717e2ab7df6b0ec20a2e64f696e547723edd991ea99d2a7963b0b182d",
                "sha256:405066a863410dfc673c63b31e312abd2d19eaa4d89810246968b8372466e9c2",
                "sha256:0f1cfb798e7ae94276a8f83a5c40d73b1650bbc12826049d1fb685c10a867d16",
                "sha256:802ab9e5c9a8721754a760a0c6c98cdc287266e3ac3fb6cb51a126886d3d987e",
                "sha256:58999b88d7d1d7ca0e87bcbd6d4709097aba82f0e22fa659b301e1c9118d92d4",
                "sha256:99ad3e9a063ca775a4b43b88983f6f25d31324b041f705b810328248e697b46a",
                "sha256:e9bdde96b6b31e985a6da55a3e67438810efc7c4681bbe118e088dd2dac3e55b",
                "sha256:e26fa9ab8c7fd408cc897c29f3dc4f3c2285625d4a9d3f0c94d5151709fde117",
                "sha256:7efa9e6341de5058a916d13c9b8580110e23858584401a062a010e4f82a372ee"
            ]
        },
        "Metadata": {
            "LastTagTime": "2021-04-16T11:40:46.654701368-04:00"
        }
    }
]
```

Show the size of the image:
```sh
docker inspect -f "$showSize" notesapp:default | numfmt --to=iec
```

![image](https://user-images.githubusercontent.com/44756128/115049597-864bc280-9ea0-11eb-80ac-7dad71aacd2b.png)

# Add a Build Stage
Open the Dockerfile:
```sh
vim Dockerfile
```

Add a build stage by adding the following:
```sh
FROM python:3 AS base
ENV PYBASE /pybase
ENV PYTHONUSERBASE $PYBASE
ENV PATH $PYBASE/bin:$PATH

FROM base AS builder
RUN pip install pipenv
WORKDIR /tmp
COPY Pipfile .
RUN pipenv lock
RUN PIP_USER=1 PIP_IGNORE_INSTALLED=1 pipenv install -d --system --ignore-pipfile

FROM base
COPY --from=builder /pybase /pybase
COPY . /app/notes
WORKDIR /app/notes
EXPOSE 80
CMD [ "flask", "run", "--port=80", "--host=0.0.0.0" ]
```

Save the file:
```sh
ESC
:wq
```

# Create a Smaller Image
Build the image:
```sh
docker build -t notesapp:multistage .
```

![image](https://user-images.githubusercontent.com/44756128/115050104-070abe80-9ea1-11eb-8c59-ce4f30dcd1f5.png)

Show the multistage image layers:
```sh
docker inspect -f "$showLayers" notesapp:multistage
```
 OUTPUT:
```
[cloud_user@ip-10-0-1-248 notes]$ docker inspect -f "$showLayers" notesapp:multistage
[
    {
        "Id": "sha256:9cc8e783d20da993f11fa791531dda63a461e9ac61c40c8b4789d79801a1d6b8",
        "RepoTags": [
            "notesapp:multistage"
        ],
        "RepoDigests": [],
        "Parent": "sha256:ece226c3c3172f7bf86f4a816d450b4d15b2c935d27d9babfd13e50ad36896f2",
        "Comment": "",
        "Created": "2021-04-16T15:46:16.012041761Z",
        "Container": "b8a22652036d7964c049eef8499a6c29106b2967749ffa3ff47802c0bdd938b8",
        "ContainerConfig": {
            "Hostname": "b8a22652036d",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/pybase/bin:/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "GPG_KEY=E3FF2839C048B25C084DEBE9B26995E310250568",
                "PYTHON_VERSION=3.9.4",
                "PYTHON_PIP_VERSION=21.0.1",
                "PYTHON_GET_PIP_URL=https://github.com/pypa/get-pip/raw/29f37dbe6b3842ccd52d61816a3044173962ebeb/public/get-pip.py",
                "PYTHON_GET_PIP_SHA256=e03eb8a33d3b441ff484c56a436ff10680479d4bd14e59268e67977ed40904de",
                "PYBASE=/pybase",
                "PYTHONUSERBASE=/pybase"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "#(nop) ",
                "CMD [\"flask\" \"run\" \"--port=80\" \"--host=0.0.0.0\"]"
            ],
            "Image": "sha256:ece226c3c3172f7bf86f4a816d450b4d15b2c935d27d9babfd13e50ad36896f2",
            "Volumes": null,
            "WorkingDir": "/app/notes",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {}
        },
        "DockerVersion": "20.10.6",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "ExposedPorts": {
                "80/tcp": {}
            },
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/pybase/bin:/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LANG=C.UTF-8",
                "GPG_KEY=E3FF2839C048B25C084DEBE9B26995E310250568",
                "PYTHON_VERSION=3.9.4",
                "PYTHON_PIP_VERSION=21.0.1",
                "PYTHON_GET_PIP_URL=https://github.com/pypa/get-pip/raw/29f37dbe6b3842ccd52d61816a3044173962ebeb/public/get-pip.py",
                "PYTHON_GET_PIP_SHA256=e03eb8a33d3b441ff484c56a436ff10680479d4bd14e59268e67977ed40904de",
                "PYBASE=/pybase",
                "PYTHONUSERBASE=/pybase"
            ],
            "Cmd": [
                "flask",
                "run",
                "--port=80",
                "--host=0.0.0.0"
            ],
            "Image": "sha256:ece226c3c3172f7bf86f4a816d450b4d15b2c935d27d9babfd13e50ad36896f2",
            "Volumes": null,
            "WorkingDir": "/app/notes",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": null
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 911600820,
        "VirtualSize": 911600820,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/02bc1653796db954081e0ca04eeec4c1842a5c2887bc93486ed8faf266b16a83/diff:/var/lib/docker/overlay2/c628d7a803b53c3c39a7003f9a16d1c9991968ff549140d7b8b65e25478462d5/diff:/var/lib/docker/overlay2/74fd90e30649aa1637ce4dc72986a772a5c40243f6c2d55937e68d4be9861f43/diff:/var/lib/docker/overlay2/f291bcae0b6035dca7683f1a5681fd7d70712ccb927e16438ef778bd0eb1818b/diff:/var/lib/docker/overlay2/c2b8df05cc93265f24fdcd683ceced81a1fce6a148bf9171a00b63d4b6bdf53c/diff:/var/lib/docker/overlay2/d8188d01f0512d08629b07446c873b7a2cdd0cd14f5c861e5253021954cd0db9/diff:/var/lib/docker/overlay2/aa6e45b45adba9e6b7c38b414eb4cf2afb576ff924a74a3abcbf063b5ed0ec34/diff:/var/lib/docker/overlay2/c069509f7873adbfe1a1d7323d977a3759555da880e9e1c44cb3f903d2fe5450/diff:/var/lib/docker/overlay2/7a136a8cbad45f23aa9d905fd9aa728b107bc359f0db8ae71ad64f5fe474da9c/diff:/var/lib/docker/overlay2/7da39d84ba6577289d5af555b29ff5500dcde7c2ac9f6817cab98801a21f3211/diff",
                "MergedDir": "/var/lib/docker/overlay2/3f15e0e40f88d5a9012d02d304db9f34283e866e56579b68ddc3ebeea88675e6/merged",
                "UpperDir": "/var/lib/docker/overlay2/3f15e0e40f88d5a9012d02d304db9f34283e866e56579b68ddc3ebeea88675e6/diff",
                "WorkDir": "/var/lib/docker/overlay2/3f15e0e40f88d5a9012d02d304db9f34283e866e56579b68ddc3ebeea88675e6/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:e2c6ff46235709f5178ab7c1939f4fba7237ffde84e13b1582fa5f0837c1d978",
                "sha256:26270c5e25fa4569f37428604f708f8863a171ec2499a066241a641017971308",
                "sha256:a42439ce96509df457152574716e3c07a8d5640fe9c16f5d4fb8854f72ce357a",
                "sha256:5d5962699bd5f016d81fc6d623cb0bc1f3853efdc48b6958053122caffc425de",
                "sha256:651326e9f1caf67559db870e293b37ca6f965da6845765b6168e86b665b2e363",
                "sha256:8d18b38717e2ab7df6b0ec20a2e64f696e547723edd991ea99d2a7963b0b182d",
                "sha256:405066a863410dfc673c63b31e312abd2d19eaa4d89810246968b8372466e9c2",
                "sha256:0f1cfb798e7ae94276a8f83a5c40d73b1650bbc12826049d1fb685c10a867d16",
                "sha256:802ab9e5c9a8721754a760a0c6c98cdc287266e3ac3fb6cb51a126886d3d987e",
                "sha256:91158965d79bcce1ec8cfe44e52bfef3da862d38ff3e2fafb3dfaf435c69778b",
                "sha256:65d4bbb71759c35d6042de1f13bbac29a18a1b217a43b345f95775eb62ed4776"
            ]
        },
        "Metadata": {
            "LastTagTime": "2021-04-16T11:46:16.032291614-04:00"
        }
    }
]

```
Count the layers:
```sh
docker inspect -f "$showLayers" notesapp:multistage | wc -l
```

Show the size of the image:
```sh
docker inspect -f "$showSize" notesapp:multistage | numfmt --to=iec
```

![image](https://user-images.githubusercontent.com/44756128/115050260-315c7c00-9ea1-11eb-87f4-e47a8520d20f.png)
