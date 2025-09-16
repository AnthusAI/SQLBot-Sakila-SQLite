# SQLBot Sakila SQLite Example

This is an example project demonstrating how to use **[SQLBot](https://github.com/AnthusAI/SQLBot)** with the Sakila sample database. The Sakila database is a well-known sample database containing DVD rental store data, perfect for learning and testing database queries.

> **New to SQLBot?** Visit the [main SQLBot repository](https://github.com/AnthusAI/SQLBot) for full documentation, installation instructions, and advanced features.

## Purpose

This project demonstrates the recommended approach for using SQLBot with your own databases:

- **Separation of Concerns**: Keep your database-specific knowledge and configuration separate from the SQLBot infrastructure code
- **Information Security**: Maintain client-confidential database information in private repositories, separate from the public SQLBot codebase
- **Customization**: Add your own database-specific agents and knowledge without modifying SQLBot itself
- **Version Control**: Track your database queries, custom agents, and project-specific configuration separately

## Progressive Enhancement Approach

**SQLBot's key strength is its zero-barrier entry with unlimited customization potential.** You can start with just a dbt profile and immediately begin exploring any database, then progressively add sophistication as your needs grow.

### Level 1: Minimal Setup (Just dbt profile)
**What you need:** Only a dbt profile in `~/.dbt/profiles.yml` or `.dbt/profiles.yml`

```yaml
Sakila:
  target: dev
  outputs:
    dev:
      type: sqlite
      path: ./profiles/Sakila/data/sakila.db
```

**What you get:**
- ✅ **Immediate functionality** - Connect and explore any database
- ✅ **Natural language queries** - "Show me the top 10 customers by rental count"
- ✅ **Automatic schema discovery** - SQLBot detects tables and columns via dbt
- ✅ **SQL assistance** - Convert natural language to proper SQL syntax

**Perfect for:** New databases, exploration, proof-of-concept, getting started

### Level 2: Schema Documentation (Optional)
**What you add:** `profiles/YourProfile/models/schema.yml` with table/column descriptions

```yaml
sources:
  - name: sakila
    tables:
      - name: film
        description: "Films available for rental"
        columns:
          - name: title
            description: "Film title"
          - name: rental_rate
            description: "Rental rate in dollars"
```

**Additional benefits:**
- ✅ **Smarter queries** - Column descriptions improve LLM understanding
- ✅ **Better suggestions** - More accurate field selection and joins
- ✅ **Team knowledge** - Shared understanding of data meaning

### Level 3: Custom Macros (Optional)
**What you add:** `profiles/YourProfile/macros/` with reusable SQL logic

```sql
{% macro top_customers(limit=10) %}
    SELECT c.customer_id, c.first_name, c.last_name, 
           COUNT(r.rental_id) as rental_count
    FROM {{ source('sakila', 'customer') }} c
    JOIN {{ source('sakila', 'rental') }} r ON c.customer_id = r.customer_id
    GROUP BY c.customer_id, c.first_name, c.last_name
    ORDER BY rental_count DESC
    LIMIT {{ limit }}
{% endmacro %}
```

**Additional benefits:**
- ✅ **Reusable logic** - Complex business calculations as simple calls
- ✅ **Consistency** - Standardized metrics across team
- ✅ **Efficiency** - "Show me top customers" → `{{ top_customers() }}`

### Level 4: Domain System Prompt (Optional)
**What you add:** `profiles/YourProfile/system_prompt.txt` with business context

```
SAKILA DVD RENTAL BUSINESS CONTEXT:
You are analyzing data from a DVD rental chain called Sakila...

KEY BUSINESS METRICS TO FOCUS ON:
- Rental volume and revenue trends
- Popular films and categories
- Customer behavior and loyalty patterns
```

**Additional benefits:**
- ✅ **Business intelligence** - Domain-aware analysis and suggestions
- ✅ **Contextual queries** - Understands industry terminology and patterns
- ✅ **Knowledge sharing** - Institutional knowledge codified and shareable
- ✅ **Onboarding** - New team members get business context automatically

### The Power of Progressive Enhancement

**Start simple, grow sophisticated:**
1. **Day 1**: Connect with just a dbt profile → immediate value
2. **Week 1**: Add schema docs → better query accuracy  
3. **Month 1**: Create macros → reusable business logic
4. **Month 3**: Add domain prompt → business intelligence

**This approach enables:**
- **Zero friction adoption** - No upfront investment required
- **Incremental value** - Each level adds meaningful capability
- **Team collaboration** - Share database knowledge through version control
- **Knowledge preservation** - Institutional knowledge survives team changes

## Project Structure

```
sqlbot-sakila-sqlite/
├── .sqlbot/              # SQLBot configuration and agents
│   ├── config.yml        # Project-specific SQLBot configuration
│   └── agents/           # Custom agents with database knowledge
├── profiles/             # Database files and configuration
│   └── Sakila/
│       └── data/
│           └── sakila.db # The actual SQLite database file
├── .dbt/                 # Local dbt profiles (created during setup)
│   └── profiles.yml      # dbt connection configuration
├── pyproject.toml        # Python project configuration
└── README.md            # This file
```

## Quick Start

### 1. Clone and Install

```bash
# Clone this example project
git clone <this-repository>
cd sqlbot-sakila-sqlite

# Install SQLBot and dependencies
pip install -e .
```

### 2. Set up the Sakila Database

Use SQLBot's built-in Sakila setup commands:

```bash
# Download the Sakila database and set up the dbt profile
sqlbot setup sakila
```

This command will:
- Download the Sakila SQLite database to `profiles/Sakila/data/sakila.db`
- Create a local `.dbt/profiles.yml` file with the proper configuration
- Verify the database installation

### 3. Start SQLBot

```bash
# Start SQLBot (profile is configured in .sqlbot/config.yml)
sqlbot
```

### 4. Try Some Queries

Once SQLBot is running, try these example queries:

- "How many films are in the database?"
- "What are the top 5 most popular film categories?"
- "Show me customers from California"
- "What's the average rental duration for films?"
- "Which actors appear in the most films?"

## Alternative Setup Commands

If you need more control over the setup process:

```bash
# Download only the database (no dbt profile setup)
sqlbot download sakila

# Set up only the dbt profile (if database already exists)
sqlbot setup sakila-profile

# Set up with custom target directory
sqlbot setup sakila --target-dir ./my-custom-path
```

## Configuration

### SQLBot Configuration

Create or modify `.sqlbot/config.yml` to customize SQLBot behavior:

```yaml
# Example configuration
llm:
  model: "gpt-5"
  max_tokens: 50000
  verbosity: "low"  # low, medium, high
  effort: "minimal"  # minimal, balanced, thorough

database:
  profile: "Sakila"
  
safety:
  read_only: true
  preview_mode: false
```

### Domain System Prompts

Add domain-specific context through profile system prompts in `.sqlbot/profiles/{ProfileName}/system_prompt.txt`:

```
# .sqlbot/profiles/Sakila/system_prompt.txt

SAKILA DVD RENTAL BUSINESS CONTEXT:
You are analyzing data from a DVD rental chain called Sakila. This is a classic sample database 
representing a video rental business from the mid-2000s.

BUSINESS OVERVIEW:
- Multi-store DVD rental chain with physical locations
- Customers rent DVDs for a few days and return them
- Revenue comes from rental fees and late fees
- Inventory management across multiple store locations
- Staff at each store handle rentals and returns

KEY BUSINESS METRICS TO FOCUS ON:
- Rental volume and revenue trends
- Popular films and categories
- Customer behavior and loyalty patterns
- Store performance comparisons
- Inventory turnover and utilization
- Staff productivity metrics

QUERY SUGGESTIONS FOR ANALYSIS:
- "Which films are most popular?" 
- "How do stores compare in revenue?"
- "What are customer rental patterns?"
- "Which categories perform best?"
- "How often do customers return late?"

Remember: This represents a traditional brick-and-mortar rental business model from before 
streaming services dominated the market.
```

**Benefits:**
- **Automatic integration** - Combined with base SQLBot system prompt
- **Business intelligence** - LLM understands domain context and terminology
- **Team knowledge sharing** - Institutional knowledge preserved in version control
- **Zero setup required** - Works immediately if file exists, ignored if missing

## Using This as a Template

### For Your Own Databases

1. **Copy this project structure**:
   ```bash
   cp -r sqlbot-sakila-sqlite my-database-project
   cd my-database-project
   ```

2. **Update pyproject.toml**:
   - Change the project name and description
   - Update dependencies if needed

3. **Replace the database**:
   - Remove `profiles/Sakila/`
   - Add your own database files to `profiles/YourDatabase/`

4. **Update dbt configuration**:
   - Modify `.dbt/profiles.yml` for your database connection
   - Use appropriate dbt adapter (postgres, snowflake, etc.)

5. **Add custom agents**:
   - Create `.sqlbot/agents/your_database_knowledge.md`
   - Document your schema, common queries, business logic

6. **Configure SQLBot**:
   - Update `.sqlbot/config.yml` with your preferences
   - Set appropriate safety and LLM settings

### Information Security Best Practices

- **Private Repositories**: Keep database-specific projects in private repositories
- **No Secrets in Code**: Use environment variables or secure vaults for credentials
- **Separate Concerns**: Don't mix client data with infrastructure code
- **Access Control**: Limit repository access to authorized team members only
- **Documentation**: Document sensitive data handling procedures

## Development

### Running Tests

```bash
# Install development dependencies
pip install -e ".[dev]"

# Run tests (if you add any)
pytest
```

### Code Formatting

```bash
# Format code
black .

# Check code style
flake8
```

## Troubleshooting

### SQLite Issues

If you encounter SQLite-related errors:

1. **Check SQLite installation**:
   ```bash
   sqlite3 --version
   ```

2. **Install SQLite** (if missing):
   - macOS: `brew install sqlite3` (or use built-in version)
   - Ubuntu/Debian: `sudo apt-get install sqlite3`
   - Windows: Download from https://sqlite.org/download.html

### dbt Profile Issues

If SQLBot can't connect to the database:

1. **Check profile configuration**:
   ```bash
   cat .dbt/profiles.yml
   ```

2. **Test dbt connection**:
   ```bash
   dbt debug --profile Sakila
   ```

3. **Verify database file exists**:
   ```bash
   ls -la profiles/Sakila/data/sakila.db
   ```

### Database Not Found

If the Sakila database wasn't downloaded properly:

```bash
# Re-run the setup
sqlbot setup sakila

# Or download manually
sqlbot download sakila
```

## About SQLBot

SQLBot is an AI-powered database interface that lets you query your databases using natural language instead of writing SQL. Built on top of dbt and SQL, it provides:

- 🗣️ **Natural Language Queries** - Ask questions in plain English
- 🔒 **Built-in Safety** - Read-only mode and query safeguards
- 🏗️ **dbt Integration** - Leverages your existing dbt models and profiles
- 🤖 **AI Agents** - Custom agents with database-specific knowledge
- 📊 **Rich Output** - Beautiful tables and data visualization

**Example queries you can try:**
- "How many films are in each category?"
- "Which actors appear in the most films?"
- "Show me customers from California"
- "What are the top 5 most rented movies?"

Learn more at **[github.com/AnthusAI/SQLBot](https://github.com/AnthusAI/SQLBot)**

## Contributing

This is an example project to demonstrate SQLBot usage patterns. For SQLBot core development:

- **Main Repository**: https://github.com/AnthusAI/SQLBot
- **Issues**: https://github.com/AnthusAI/SQLBot/issues
- **Documentation**: https://github.com/AnthusAI/SQLBot/wiki

## License

This example project is provided under the MIT License. The Sakila database is provided by MySQL under a BSD license.

---

**Need Help?** 

- Check the [SQLBot documentation](https://github.com/AnthusAI/SQLBot)
- Open an issue on the [SQLBot repository](https://github.com/AnthusAI/SQLBot/issues)
- Join our community discussions