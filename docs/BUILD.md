# Build and Release

## Local Build

TunnelX currently targets 64-bit Windows only. The project file sets `PlatformTarget` to `x64`, and the bundled native components under `AppTunnel/NativeLibs/x64` are required for the current build.

Building from source requires the .NET 8 SDK. Running a framework-dependent developer build requires the .NET 8 Desktop Runtime or SDK on the machine.

```powershell
dotnet build AppTunnel.sln -c Release
```

## Standalone Compressed EXE

This is the recommended public release format. It is self-contained, so users do not need to install .NET 8 separately. Native components are bundled and extracted by the app at runtime when needed.

```powershell
dotnet publish AppTunnel\AppTunnel.csproj -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true -p:EnableCompressionInSingleFile=true -p:IncludeNativeLibrariesForSelfExtract=true -p:DebugType=None -p:DebugSymbols=false -o publish\TunnelX-standalone-compressed-exe
```

Rename the final executable with the app version:

```powershell
TunnelX-v1.2.23-standalone-compressed.exe
```

## GitHub Actions Release

Public releases are published by `.github/workflows/release.yml`.

The release workflow:

- runs on `windows-latest` with .NET 8;
- accepts tags in `vMAJOR.MINOR.PATCH` format, such as `v1.2.23`;
- verifies that the tag version matches `<Version>` in `AppTunnel/AppTunnel.csproj`;
- builds and publishes the `win-x64` self-contained single-file executable;
- attaches `TunnelX-vX.Y.Z-standalone-compressed.exe` and a `.sha256` checksum to the GitHub Release.

To publish a release from the command line:

```powershell
git tag v1.2.23
git push origin v1.2.23
```

The same workflow can also be run manually from GitHub Actions by providing the release tag.

## 32-bit Windows

32-bit Windows builds are not supported at this time. Supporting `win-x86` would require a separate compatibility pass, x86-compatible native binaries for every bundled network component, and separate testing for WinDivert/Wintun, Xray/sing-box, packet interception, route management, and the standalone extraction path.

## Before Publishing

- Run leak, DNS, full-route, split-route, app toggle, and reconnect tests before attaching a public artifact.
- Confirm third-party license notices are current.
- Confirm the app version in `AppTunnel/AppTunnel.csproj`.
- Attach release artifacts only to GitHub Releases; do not commit generated `publish/` or `Releases/` output.

## Missing Runtime Behavior

The app cannot show a custom .NET missing-runtime message if it is built as framework-dependent and the target machine does not have the required .NET Desktop Runtime, because the .NET host fails before TunnelX code starts. For public releases, publish the self-contained standalone EXE instead of relying on runtime installation. If an installer is added in the future, the installer/bootstrapper can check and install prerequisites before launching the app.
