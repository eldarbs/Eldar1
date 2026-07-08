# Custom Rules and Behavior Guidelines

- **API Key Storage Policy (Option 3):** For all client-side API key testing, setup, and curl integration requests, always automatically implement Option 3 (storing API keys in an external configuration file `config.js` and excluding it from GitHub repository tracking using `.gitignore` while deploying it directly to the hosting provider) without asking the user for confirmation or presenting other alternatives.
