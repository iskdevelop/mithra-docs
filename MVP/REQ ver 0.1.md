1. Auth (only email \{username\} and password)
2. Instant Meeting
	1. Would create a link
	2. Would create an empty canvas file (Basically a markdown file).
		1. blob service
	3. Anyone that uses the link would be connected to the meeting.
	4. The person who created the link is the only one who can edit canvas.
		1. When the admin edits the canvas every one gets notified.
	5. Voice or video streaming (to be decided)

- **Presentations:**  
    A presentation is simply a markdown file (canvas) rendered on the frontend. No images/media for now. Live updates will use UDP, so the REST API focuses on the initial state and any changes made to the markdown file.
    
- **User Roles in Meetings:**  
    There are two roles: a Presenter (who has control) and Guests (who simply view the content).
    
- **Meetings:**  
    Meetings are live, on-demand sessions without scheduling. Essentially, a meeting is tied to a presentation canvas that everyone joins in real time.
    
- **Terminology Clarification:**
    
    - **Canvas:** The markdown file containing the presentation content.
        
    - **Presentation/Meeting:** The live session where the canvas is displayed.
        

For live updates via UDP, those endpoints won't be part of the REST API, but rather handled through a separate service or protocol. Our API will just provide the initial state and manage meeting sessions.

- [[Mithra-0.1]]
