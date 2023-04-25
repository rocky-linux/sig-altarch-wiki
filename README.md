# SIG/AltArch Wiki

This is a Wiki for the Rocky Linux Alternative Architectures SIG.  It's an information repository that records the group efforts and findings.

The AltArch special interest group looks at porting Rocky Linux to diverse or non-traditional platforms.  Think small boards (Raspberry Pi and relatives), or other processor architectures like RISC-V and armhfp (32-bit ARM).


This Wiki is browseable at **https://sig-altarch.rocky.page** .



## Contributing

Feel free to contribute to this Wiki via pull request.

You can test any changes locally by running it with Docker/Podman compose on your PC.  Clone this wiki, `cd` into the folder, and run:

```
docker-compose up
```

It should build and run the wiki container.  You can browse it and see your live edits by visiting `localhost:8000` .



## Continuous Integration / Continuous Deployment

Actions Runner executes workflow to publish to https://sig-altarch.rocky.page on push to main.
