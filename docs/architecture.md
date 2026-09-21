# Cyber Operations Console — Application Architecture

## Architecture Goals
- Clearly separate the responsibilities of the frontend, backend, and PCAP analysis engine so that each component has a focused purpose.

- Keep the frontend loosely coupled to the backend by defining a stable interface and data contract. The frontend should depend on the structure of the application data it receives rather than the backend's internal implementation.

- Normalize PCAP analysis results into a consistent and predictable format that the frontend can reliably use to display capture summaries, access points, stations, relationships, and other analysis data.

- Design file processing so that large packet captures can be analyzed without blocking or freezing the application's user interface.

## System Components

### Frontend
- Present analysis data to the user in a clear, concise, and visually organized manner.
- Allow the user to interact with capture data through high-level summaries, access point and station views, filtering, and drill-down interactions.
- Provide clear and consistent navigation between major areas of the application.
- Provide the interface for selecting and uploading supported packet-capture files.
- Manage frontend application state such as the active capture, selected access point or station, current filters, and loading or error states.
- Provide clear feedback based on information received from the backend, including processing status, successful analysis, unavailable data, validation problems, and errors.

### Backend

- Receive packet-capture uploads and other requests from the frontend.
- Validate uploaded files, request data, and supported file types before analysis begins.
- Coordinate the analysis workflow by passing valid capture files to the PCAP analysis engine and tracking the analysis process.
- Receive analysis results from the PCAP analysis engine and prepare them for use by the rest of the application.
- Expose structured, predictable application data to the frontend through a defined API and data contract.
- Return appropriate processing states, validation messages, and errors so the frontend can provide clear feedback to the user.

### PCAP Analysis Engine

- Read and process supported PCAP and PCAPNG files provided by the backend.

- Interpret relevant packet and wireless frame structures contained within the capture.

-  Identify observed wireless access points and stations from the available packet evidence.

- Determine observable relationships between access points and stations and preserve uncertainty when a relationship cannot be confidently established.

- Generate useful capture statistics and analysis results for use by the application.

- Transform raw packet-level information into normalized, intelligible application data that can be returned through the backend to the frontend.

## Data Flow
- The user selects and uploads a supported PCAP or PCAPNG file through the frontend.
- The frontend sends the capture file to the backend through the application's API.
- The backend validates the request and uploaded file before allowing analysis to continue.
- The backend passes the validated capture to the PCAP analysis engine.
- The PCAP analysis engine parses the capture, interprets relevant packet and wireless frame data, and extracts information about the capture, access points, stations, and observed relationships.
- The PCAP analysis engine converts the raw packet evidence into normalized analysis results and returns those results to the backend.
- The backend prepares the analysis results according to the application's API and data contract.
- The backend sends the structured results to the frontend.
- The frontend stores the received application data in its current state and presents the analysis to the user through the appropriate overview, access point, station, and relationship views.

## Data Boundary
- The boundary between the PCAP analysis engine and the rest of the application should separate raw packet data from normalized application data.

- The PCAP analysis engine is responsible for understanding packet-level structures, wireless frame fields, protocol details, MAC addresses, and other low-level capture information. These internal parsing details should remain inside the analysis layer unless they are specifically needed for an advanced drill-down feature.

- The analysis engine should convert relevant packet evidence into predictable application entities such as:

* CaptureSummary
* AccessPoint
* Station
* Relationship
* AnalysisWarning

- The backend should expose these normalized entities through a stable API and data contract. The frontend should rely on this structured data rather than depending on the internal implementation of the PCAP parser.

- This boundary should allow the packet-analysis implementation to change in the future without requiring major changes to the frontend, as long as the agreed data contract remains compatible.

## Initial Technology Direction

The Cyber Operations Console will use a separated frontend, backend API, and PCAP analysis architecture so that each major component can evolve independently while communicating through a stable data contract.

### Frontend

The frontend will be developed using React with TypeScript and Vite.

React will provide a component-based structure for building the interactive analysis interface, while TypeScript will provide stronger guarantees around the structured application data received from the backend. Vite will provide the frontend development and production build tooling.

Styling will use CSS Modules and CSS custom properties. This will allow the application to develop a reusable design system while maintaining clear ownership and organization of component styles.

### Backend

The backend will use Python and FastAPI.

FastAPI will provide the HTTP API responsible for file uploads, request validation, analysis coordination, processing status, error handling, and delivery of normalized analysis results to the frontend.

Pydantic models will define and validate the application's data contract, including entities such as capture summaries, access points, stations, relationships, and analysis warnings.

### PCAP Analysis Engine

The initial PCAP analysis engine will be implemented in Python using Scapy.

Scapy will be responsible for reading PCAP and PCAPNG captures and providing access to relevant packet and IEEE 802.11 frame information.

Application-specific analysis logic will operate above the packet parsing layer to identify access points, stations, relationships, and capture statistics and transform that evidence into normalized application data.

The analysis engine will be designed so that additional analysis implementations, including tools such as TShark, can be incorporated later without changing the frontend data contract.

### Analysis Processing

Capture analysis should be treated as a potentially long-running operation.

The architecture should allow the backend to accept an analysis request, track its processing state, and allow the frontend to remain responsive while analysis is performed.

The initial implementation should avoid unnecessary distributed infrastructure, while preserving the ability to introduce dedicated background workers and job queues if capture size or processing requirements later require them.

### Persistence

Persistent storage is not required for the initial MVP.

Database storage should be introduced only when features such as saved projects, persistent analysis sessions, users, historical captures, or reporting require it.
