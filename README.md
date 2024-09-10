# Misskey Emoji Bulk Uploader

## Overview

The Misskey Emoji Bulk Uploader is a web application built with Nuxt 3 that allows users to easily upload multiple custom emojis to their Misskey instance. This tool streamlines the process of adding new emojis, making it efficient for server administrators and emoji enthusiasts.

## Features

- Bulk upload of multiple emoji files
- Support for various image formats (PNG, APNG, GIF, etc.)
- Custom naming for each emoji
- Category assignment for uploaded emojis
- Configurable rate limiting to prevent server overload
- Pause and resume functionality for uploads
- Individual and bulk removal of selected emojis
- Lazy loading of emoji previews for improved performance
- Dark theme user interface

## Prerequisites

- Node.js (v14 or later recommended)
- npm or yarn
- A Misskey instance with API access

## Installation

1. Clone the repository:
   ```
   git clone https://github.com/yourusername/misskey-emoji-bulk-uploader.git
   cd misskey-emoji-bulk-uploader
   ```

2. Install dependencies:
   ```
   npm install
   # or
   yarn install
   ```

3. Build the application:
   ```
   npm run build
   # or
   yarn build
   ```

## Usage

1. Start the application:
   ```
   npm run start
   # or
   yarn start
   ```

2. Open a web browser and navigate to `http://localhost:3000` (or the appropriate address if you've configured it differently).

3. Enter your Misskey instance URL and API key. Make sure to use an API key with emoji addition permissions.

4. (Optional) Enter a category for the emojis you're uploading.

5. Set a rate limit if desired (0 is recommended if your role allows it).

6. Select the emoji files you want to upload.

7. (Optional) Customize the names of the emojis in the preview grid.

8. Click "Upload Emojis" to start the upload process.

9. Use the "Pause" and "Resume" buttons to control the upload process as needed.

## Important Notes

- Ensure that your API key has the necessary permissions to add emojis to your Misskey instance.
- The application replaces hyphens with underscores in emoji names for compatibility.

## Contributing

Contributions to the Misskey Emoji Bulk Uploader are welcome! Please feel free to submit pull requests, create issues, or suggest new features.

## Acknowledgements

- This project uses [Nuxt 3](https://nuxt.com/), [Tailwind CSS](https://tailwindcss.com/), and [DaisyUI](https://daisyui.com/).
- Special thanks to the Misskey community for inspiration and support.
