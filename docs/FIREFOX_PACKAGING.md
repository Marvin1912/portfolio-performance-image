# Firefox Packaging Documentation

## Overview

The Portfolio Performance Docker container supports a Firefox packaging option that transforms it from a simple web-based interface into a full desktop environment. This is particularly useful when the application needs to open a browser for authentication processes.

## What Firefox Packaging Does

### Installed Components

When building with `--build-arg PACKAGING=firefox`, the following components are installed:

1. **Firefox ESR** - Extended Support Release of Firefox browser
2. **XFCE4** - Lightweight desktop environment
3. **Thunar** - XFCE file manager
4. **Tint2** - Taskbar/panel for the desktop environment
5. **DBus & DBus-X11** - System communication bus for desktop applications
6. **GSimpleCal** - Calendar application
7. **Inotify-tools** - File system monitoring utilities
8. **Cron** - Task scheduler

### Configuration Changes

The Firefox packaging makes several configuration changes:

#### Firefox Profile Setup
- Creates a default Firefox profile at `/root/.mozilla/firefox`
- Sets proper permissions (`chmod -R 777`) for the profile directory

#### Firefox Policies
A policy file is created at `/usr/lib/firefox-esr/distribution/policies.json` with the following settings:
- **Homepage**: Sets DuckDuckGo as the default homepage
- **First Run Page**: Overrides first run page to DuckDuckGo
- **Extensions**: Automatically installs uBlock Origin ad-blocker

#### Taskbar Configuration (Tint2)
The taskbar is extensively configured:
- Removes unavailable application launchers to prevent error messages
- Activates auto-hide functionality with 10px height when hidden
- Sets immediate show timeout (0.0s) and 2-second hide timeout
- Adds file manager and Firefox launchers to the taskbar
- Changes mouse actions: middle-click toggles, right-click closes

## How It Solves Browser Authentication

### The Problem
Portfolio Performance sometimes needs to open a browser for OAuth authentication when connecting to financial services. In a standard container setup, there's no browser available inside the container.

### The Solution
With Firefox packaging:
1. Portfolio Performance can launch Firefox inside the container
2. Authentication happens within the containerized Firefox instance
3. Users interact with Firefox through the web-based desktop interface
4. No need for host OS browser integration

## Usage

### Building with Firefox Support

```bash
docker build --build-arg PACKAGING=firefox -t portfolio-performance:firefox .
```

### Running the Container

The container can be run normally with docker-compose:

```yaml
version: "3"
services:
  portfolio-performance:
    image: portfolio-performance:firefox
    container_name: portfolio
    restart: unless-stopped
    ports:
      - 5800:5800
    volumes:
      - /opt/docker-volumes/pp/config:/config
      - /opt/docker-volumes/pp/workspace:/opt/portfolio/workspace
    environment:
      USER_ID: 1000
      GROUP_ID: 1000
      DISPLAY_WIDTH: 1920
      DISPLAY_HEIGHT: 1080
      TZ: "Europe/Berlin"
```

### Accessing the Desktop

1. Open your web browser and navigate to `http://localhost:5800`
2. You'll see the XFCE desktop environment
3. Firefox can be launched from the taskbar or by Portfolio Performance
4. Complete authentication workflows within the containerized Firefox

## Technical Details

### File System Locations
- **Firefox Profile**: `/root/.mozilla/firefox/`
- **Firefox Policies**: `/usr/lib/firefox-esr/distribution/policies.json`
- **Taskbar Config**: `/usr/share/tint2/vertical-neutral-icons.tint2rc`

### Environment Variables
The standard jlesage/baseimage-gui environment variables are respected:
- `DISPLAY_WIDTH`: Desktop width (default: 1920)
- `DISPLAY_HEIGHT`: Desktop height (default: 1080)
- `WEB_LISTENING_PORT`: Web interface port (default: 5800)

### Browser Integration
Portfolio Performance automatically detects and uses the Firefox installation when available. No additional configuration is required.

## Comparison with Other Solutions

| Solution | Pros | Cons |
|----------|------|------|
| Firefox Packaging | Self-contained, no host dependencies, works through web interface | Larger image size, additional desktop components |
| Host X11 Forwarding | Uses host browser, familiar environment | Complex setup, security considerations |
| Manual Auth | Simple, no changes needed | Manual process, error-prone |

## Troubleshooting

### Firefox Not Available
Ensure the image was built with `PACKAGING=firefox`:
```bash
docker exec portfolio firefox-esr --version
```

### Authentication Issues
Check that Firefox can access external websites:
```bash
docker exec portfolio firefox-esr --headless https://example.com
```

### Performance Considerations
Firefox packaging increases image size by approximately 500MB. Consider using only if browser authentication is required.

## Security Notes

- Firefox runs as root inside the container
- All browser activity is contained within the container
- External network access is still governed by Docker networking rules
- Consider the security implications of running a full desktop environment in containers