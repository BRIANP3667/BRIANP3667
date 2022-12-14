name: publish
on:
  push:
    tags:
       - '*'
jobs:
  docker:
    runs-on: ubuntu-latest
    env:
      REPOSITORY: unblockneteasemusic
      DOCKER_USERNAME: nondanee
      DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
    steps:
      -
        name: Prepare
        id: prepare
        run: |
          ARCH=(amd64 arm/v6 arm/v7 arm64 386 ppc64le s390x)
          PLATFORM=$(printf ",linux/%s" "${ARCH[@]}")
          echo ::set-output name=build_platform::${PLATFORM:1}
          echo ::set-output name=image_name::${DOCKER_USERNAME}/${REPOSITORY}
      -
        name: Checkout
        uses: actions/checkout@v2
      -
        name: Setup Buildx
        uses: crazy-max/ghaction-docker-buildx@v1
      -
        name: Login
        run: |
          echo "${DOCKER_PASSWORD}" | docker login --username ${DOCKER_USERNAME} --password-stdin
      -
        name: Build
        run: |
          docker buildx build \
            --tag ${{ steps.prepare.outputs.image_name }} \
            --platform ${{ steps.prepare.outputs.build_platform }} \
            --output "type=image,push=true" \
            --file Dockerfile .
      -
        name: Check Manifest
        run: |
          docker buildx imagetools inspect ${{ steps.prepare.outputs.image_name }}
  npm:
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v2
      -
        name: Setup Node.js
        uses: actions/setup-node@v1
        with:
          registry-url: https://registry.npmjs.org/
      -
        name: Publish
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
