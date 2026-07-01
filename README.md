# Remote_Video_Capture_Portal
A FastAPI-driven backend to ingest live remote video feeds, manage in-memory metadata, and optimize cross-device HTTPS 206 streaming for downstream AI-driven analytical pipelines.

# How It Works

The system is built around asynchronous Python modules and integrated HTML5 interfaces that run in two primary phases: video ingestion and optimized streaming.
Hardware Abstraction: The frontend queries the host device using WebRTC constraints, prioritizing the environment (back-facing) camera for high-fidelity data capture.
Secure Ingestion Interface (Frontend): Leverages the native MediaRecorder API to capture live .webm video from the selected hardware and transmits it via chunked HTTP POST requests.
Secure Uvicorn/FastAPI Server (Backend): Orchestrates the asynchronous pipeline. It utilizes self-signed SSL certificates to enforce HTTPS, ensuring secure communication within the local network (LAN).
Storage Pipeline: Streams incoming video data in 1MB chunks directly to local disk storage (video_storage/) to prevent Out-Of-Memory (OOM) failures on the host machine.
In-Memory Database (video_db): Extracts file metadata (IDs, sizes, paths) and indexes them in a rapid-access RAM dictionary for extremely fast lookup speeds.

# System Workflow

Phase 1 - Video Generation & Ingestion

A remote user accesses the secure IP-based URL via a mobile or desktop browser over the local network.
The system requests hardware permissions and defaults to the high-quality environment camera.
The user captures live video, which is compiled into a Blob payload by the browser and transmitted to the /api/v1/upload/ endpoint.
The system writes the file to disk and simultaneously registers its metadata in the in-memory database.

Phase 2 - Video Consumption

A user navigates to the /gallery interface on a separate device.
The system fetches the in-memory metadata list to populate the interface.
When playback is requested, the /api/v1/stream/{id} endpoint delivers the file using HTTP 206 Partial Content rules, enabling scrubbable, chunked playback across any device or operating system.
Prerequisites
Python 3.11+
Devices must be operating on the same secure Local Area Network (LAN) or Corporate VPN.
A device with at least one video input (built-in webcam, USB camera, or mobile camera).
Project Setup
1. Open the Project Environment Ensure the runtime environment is operating out of a dedicated project directory.
2. Create a Virtual Environment
Bash
python -m venv venv
source venv/bin/activate  # macOS / Linux

# Install Dependencies

Bash
pip install fastapi uvicorn python-multipart nest-asyncio pyngrok
6. Launch the Application Pipeline Run the Uvicorn server configuration. Upon startup, the system will execute a disk-recovery sequence to rebuild the in-memory database from existing local files before accepting network traffic.

# Host Environment/
+-- video_storage/ (Physical disk storage for .webm/.mp4)
+-- Main Application/
    +-- GET / (Live Recording UI & Hardware Selection)
    +-- GET /gallery (Playback UI)
    +-- POST /api/v1/upload/ (Ingestion routing)
    +-- GET /api/v1/videos/ (Metadata dictionary fetch)
    +-- GET /api/v1/stream/{video_id} (Chunked media delivery)

# Technologies Used

Category           |     Technology
Language           |     Python 3.11
Backend Framework  |     FastAPI / Uvicorn
Database (Cache)   |     Python In-Memory Dictionary
Frontend Rendering |     HTML5, CSS (Inter), WebRTC

# System Outputs

Centralized Media Storage: Raw video files are securely deposited onto the host machine's disk, ready for downstream processing.
Indexed Metadata: Instantly queryable JSON structures containing file paths, sizes, and MIME types.
Cross-Device Stream: Media dynamically delivered over public connections without downloading the entire source file.
Key Design Decisions
Asynchronous Architecture: Built on standard asyncio to ensure video I/O processes do not block UI rendering or concurrent uploads.
Secure LAN Deployment: Enforced HTTPS via self-signed SSL certificates (key.pem, cert.pem), ensuring WebRTC camera access is granted by modern browsers within a secure local environment.
Memory-Optimized Processing: File payloads are processed in 1MB chunked streams to maintain low RAM utilization regardless of input file size.
State Persistence: Integrated an OS-level directory scan upon server boot to automatically repopulate the volatile in-memory database with existing disk assets.
Protocol Compliance: Utilized FileResponse for streaming to ensure strict adherence to HTTP 206 Partial Content requirements enforced by WebKit-based browsers.


# Future Enhancements

Integration of OpenCV and face_recognition libraries to process incoming .webm frames for identity verification.
Migration of local disk storage to cloud-based enterprise object storage (e.g., AWS S3 or Azure Blob).
Migration of the in-memory dictionary to a persistent relational database like PostgreSQL.
Containerization of the FastAPI application via Docker for scalable deployment.

# Conclusion

This PoC demonstrates how modern Python async frameworks can automate the ingestion of heavy media files across distributed local networks. By separating frontend capture mechanisms from a high-concurrency backend, this system provides a robust, scalable foundation for subsequent facial recognition pipelines, allowing for the integration of unstructured media into a highly organized, queryable enterprise environment.

