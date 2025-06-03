# goes-data-explorer
Search and Download data from GOES-R series satellites

This repository contains a single-page web application for exploring and downloading data from the AWS Open Data buckets for the GOES-R series satellites.

This application is also deployed and accessible via GitHub Pages:
[https://coliveir-aer.github.io/goes-data-explorer/](https://coliveir-aer.github.io/goes-data-explorer/)

**AWS Open Data Link:** [GOES on AWS Marketplace](https://aws.amazon.com/marketplace/pp/prodview-ngejrbcumyjtu?sr=0-33&ref_=beagle&applicationId=AWSMPContessa)

All data queries are executed as client-side JavaScript requests directly against the AWS S3 APIs. Individual files are then downloaded into the user's default download folder, bypassing any server-side processing for data transfer.

---

## Features

* **Satellite Selection:** Choose between GOES-16, GOES-17, GOES-18, and GOES-19.
* **Product Type Filtering:** Select from a comprehensive list of GOES-R ABI, GLM, and SUVI product types.
* **Coverage Region:** Filter data by Full Disk (FD), CONUS (C), Mesoscale 1 (M1), or Mesoscale 2 (M2) for ABI products.
* **Channel Selection (ABI):** For relevant ABI products (L1b-Rad, CMIP, DMW, DMWV), select specific channels (01-16) to refine your search.
* **Date and Time Specification:** Pinpoint data for a specific date (UTC) and hour/minute (UTC).
* **S3 Prefix Generation:** Automatically generates the S3 prefix used for queries, aiding in understanding the data structure.
* **Client-Side S3 Querying:** Directly lists available files from AWS S3 buckets using JavaScript.
* **Individual File Download:** Users can select and download multiple files directly to their local machine.
* **Select/Deselect All:** Convenient buttons for managing multiple file selections.

---

## Getting Started

### Prerequisites

You only need a modern web browser to use the GOES Data Explorer. No special accounts, AWS credentials, or server-side setup are required for basic usage, as it interacts with publicly available AWS Open Data.

### Running Locally

To run this application locally:

1.  **Clone the repository:**
    ```bash
    git clone [https://github.com/coliveir-aer/goes-data-explorer.git](https://github.com/coliveir-aer/goes-data-explorer.git)
    cd goes-data-explorer
    ```
2.  **Open `index.html`:**
    Simply open the `index.html` file in your preferred web browser. Due to the client-side nature of the application, it runs directly from your file system.

### Deployment

To deploy this application to a web server:

1.  Upload the `index.html` file (and any associated CSS/JS files if they were externalized) to any standard web server (e.g., Apache, Nginx, GitHub Pages, AWS S3 with static website hosting enabled).
2.  Ensure that the server is configured to serve static HTML files.


---

## For Developers

This section provides a deeper dive for developers interested in understanding, modifying, or extending the GOES Data Explorer.

### Project Structure

The project consists of a single HTML file:

* `index.html`: Contains all the HTML structure, CSS styling (via Tailwind CSS CDN and inline styles), and JavaScript logic.

### Technologies Used

* **HTML5:** For the page structure.
* **Tailwind CSS (CDN):** For utility-first CSS styling, enabling rapid UI development.
* **Vanilla JavaScript:** All interactive logic, S3 API interactions, and DOM manipulation are handled with pure JavaScript, without frameworks.

### AWS S3 Interaction

The core functionality relies on direct client-side requests to AWS S3.

1.  **S3 ListObjectsV2 API:**
    * The application constructs an S3 prefix based on user selections (satellite, product type, date, time, region, and channels).
    * It then makes `GET` requests to the S3 bucket's regional endpoint (e.g., `https://noaa-goes19.s3.us-east-1.amazonaws.com/?list-type=2&prefix=...`) to list objects within that prefix.
    * The `list-type=2` parameter is used for the newer version of the List Objects API, which supports continuation tokens for paginated results.
    * Responses are XML, which are then parsed using `DOMParser` to extract object keys (`<Key>`).
    * **Pagination:** The application handles S3 API pagination by checking the `<IsTruncated>` tag and using `<NextContinuationToken>` to make subsequent requests until all results are fetched.

2.  **S3 Object Download:**
    * When a user clicks "Download Selected Files," the application constructs a direct HTTPS URL for each selected S3 object (e.g., `https://bucket-name.s3.region.amazonaws.com/path/to/key`).
    * It then programmatically creates a temporary `<a>` element, sets its `href` to the download URL, `target` to `_blank`, and `download` attribute to the filename, and programmatically `clicks()` it. This triggers the browser's native download mechanism.

### Key JavaScript Functions

* `getCurrentS3Prefix()`: Dynamically constructs the S3 prefix string based on the current UI selections. This function correctly formats prefixes for ABI (with region suffixes), GLM, and SUVI products, and calculates the day of the year.
* `toggleChannelSelector()`: Manages the visibility and enabled state of the ABI channel selection checkboxes based on the selected product type. It also handles the visibility of the coverage region radios for ABI vs. non-ABI products.
* `queryButton.addEventListener('click', async () => { ... })`: The main event handler for initiating S3 queries. It handles:
    * Input validation.
    * Construction of the S3 List Objects API URL.
    * Fetching and parsing XML responses.
    * Handling S3 pagination (`IsTruncated`, `NextContinuationToken`).
    * Filtering results for Mesoscale regions and specific ABI channels.
    * Updating the results display.
    * Error handling and loading indicators.
* `downloadButton.addEventListener('click', async () => { ... })`: Handles the download of selected files by programmatically creating and clicking `<a>` elements for each selected S3 key.
* `selectAllButton.addEventListener('click', () => { ... })`: Toggles the checked state of all file selection checkboxes in the results area.


### Contributing

Contributions are welcome! If you have suggestions for improvements, bug fixes, or new features, please open an issue or submit a pull request.

1.  Fork the repository.
2.  Create a new branch (`git checkout -b feature/your-feature-name`).
3.  Make your changes.
4.  Commit your changes (`git commit -m 'Add new feature'`).
5.  Push to the branch (`git push origin feature/your-feature-name`).
6.  Open a Pull Request.
