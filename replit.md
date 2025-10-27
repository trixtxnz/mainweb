# Overview

This is a Flask-based web application that provides user authentication, personalized user experiences, and a simple clicker game. The application allows users to register, log in, customize their homepage preferences (welcome text, colors, font size, background image), and play a clicker game that tracks their clicks. User data is persisted in a JSON file, and the application uses session-based authentication with file upload capabilities for custom backgrounds.

# User Preferences

Preferred communication style: Simple, everyday language.

# System Architecture

## Backend Architecture

**Framework**: Flask web framework with session-based authentication

**Data Storage**: File-based JSON storage (`users.json`) for user accounts and preferences
- **Problem**: Need persistent storage for user data without database complexity
- **Solution**: Simple JSON file with load/save utilities
- **Pros**: No database setup required, easy to inspect and debug
- **Cons**: Not suitable for concurrent writes, scalability limitations, no ACID guarantees

**Authentication**: SHA-256 password hashing with server-side sessions
- **Problem**: Secure user authentication and password storage
- **Solution**: Hash passwords with SHA-256 before storage, use Flask sessions for authentication state
- **Pros**: Simple implementation, passwords not stored in plaintext
- **Cons**: SHA-256 alone lacks salt, vulnerable to rainbow table attacks (should use bcrypt/argon2 in production)

**User Data Model**: Each user object contains:
- `password`: Hashed password string
- `gender`: User gender field
- `clicks`: Click counter for the clicker game
- `click_bonus`: Stacking multiplier for clicks earned per click (starts at 1)
- `has_unlocked_100`: Boolean flag tracking if user has unlocked the 100 clicks upgrade tier
- `has_unlocked_1000`: Boolean flag tracking if user has unlocked the 1000 clicks upgrade tier
- `prefs`: Dictionary of customization preferences (welcome_text, bg_color, text_color, font_size, bg_image)

**File Upload System**: Background image uploads with security features
- **Problem**: Allow users to personalize their page with custom background images
- **Solution**: Secure file upload with validation, sanitization, and automatic cleanup
- **Security**: Username validation prevents path traversal; secure_filename sanitization; file type restrictions; 16MB size limit
- **Storage**: Images saved to `static/uploads/` with format `{username}_bg.{ext}`
- **Cleanup**: Automatic deletion of old images when replaced or removed

## Frontend Architecture

**Template Engine**: Jinja2 (Flask's default templating)

**Pages**:
- `index.html`: Landing/login page
- `welcome.html`: Post-login welcome page
- `website.html`: Personalized user homepage with dynamic styling based on preferences
- `settings.html`: User preference customization interface
- `clicker game.html`: Interactive clicker game with tiered upgrade system:
  - Tier 1: Spend 10 clicks for +1 click bonus (unlocks tier 2)
  - Tier 2: Spend 100 clicks for +5 click bonus (unlocks tier 3)
  - Tier 3: Spend 1000 clicks for +10 click bonus
  - Visual feedback: buttons are grayed out when user can't afford them
  - Progressive unlocking: higher tiers only appear after purchasing lower tiers

**Styling Approach**: Inline CSS in templates with dynamic values injected from user preferences
- **Problem**: Need personalized styling per user
- **Solution**: Jinja2 template variables for CSS properties (background color, text color, font size)
- **Pros**: Simple, no build process required
- **Cons**: Limited reusability, harder to maintain consistent global styles

## Application Configuration

**Secret Key**: Hardcoded secret key for session management (flagged for production change)
- Note: Should be moved to environment variables for production deployment

**Session Management**: Server-side sessions using Flask's built-in session handling

# External Dependencies

## Python Packages
- **Flask** (>=3.1.2): Web framework for routing, templating, and request handling
- **Werkzeug** (>=3.1.3): WSGI utility library (Flask dependency)

## Storage
- **JSON File System**: `users.json` for persistent user data storage
  - Contains user credentials, preferences, and game state
  - No external database required

## Standard Library Dependencies
- **json**: For parsing and serializing user data
- **os**: For file system operations
- **hashlib**: For SHA-256 password hashing

Note: The application currently has no external API integrations, third-party services, or database systems. All data is self-contained within the application file system.