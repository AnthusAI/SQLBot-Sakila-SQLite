# Sakila Database Knowledge

This agent provides comprehensive knowledge about the Sakila database schema, relationships, and common query patterns.

## Database Overview

The Sakila database is a sample database representing a DVD rental store. It contains information about films, actors, customers, rentals, payments, and store operations.

## Core Tables and Relationships

### Films and Catalog
- **film**: Contains movie information (title, description, release_year, rating, length, etc.)
- **category**: Film categories (Action, Comedy, Drama, etc.)
- **film_category**: Links films to categories (many-to-many)
- **language**: Film languages
- **film_text**: Full-text searchable film descriptions

### Actors and Cast
- **actor**: Actor information (first_name, last_name)
- **film_actor**: Links actors to films (many-to-many relationship)

### Customers and Geography
- **customer**: Customer information (names, email, address_id, store_id, active status)
- **address**: Street addresses linked to cities
- **city**: Cities linked to countries
- **country**: Country information

### Rental Business Logic
- **rental**: Rental transactions (rental_date, return_date, customer_id, inventory_id, staff_id)
- **inventory**: Physical copies of films at stores (film_id, store_id)
- **payment**: Payment records (amount, payment_date, customer_id, rental_id, staff_id)

### Store Operations
- **store**: Store locations (address_id, manager_staff_id)
- **staff**: Employee information (names, email, store_id, active status)

## Important Business Rules

### Rental Duration
- Each film has a `rental_duration` (usually 3-7 days)
- Late returns may incur additional fees
- Rental rate varies by film

### Inventory Management
- Multiple copies of popular films exist across stores
- `inventory_id` tracks individual copies
- Films can be unavailable if all copies are rented

### Payment Structure
- Payments are typically made at rental time
- Late fees may generate additional payment records
- Amount includes base rental fee plus any applicable taxes/fees

### Customer Status
- Customers can be marked as inactive
- Active customers can rent films
- Customer addresses determine store assignment

## Common Query Patterns

### Popular Films Analysis
```sql
-- Most rented films
SELECT f.title, COUNT(r.rental_id) as rental_count
FROM film f
JOIN inventory i ON f.film_id = i.film_id
JOIN rental r ON i.inventory_id = r.inventory_id
GROUP BY f.film_id, f.title
ORDER BY rental_count DESC;
```

### Revenue Analysis
```sql
-- Total revenue by film
SELECT f.title, SUM(p.amount) as total_revenue
FROM film f
JOIN inventory i ON f.film_id = i.film_id
JOIN rental r ON i.inventory_id = r.inventory_id
JOIN payment p ON r.rental_id = p.rental_id
GROUP BY f.film_id, f.title
ORDER BY total_revenue DESC;
```

### Customer Analysis
```sql
-- Most active customers
SELECT c.first_name, c.last_name, COUNT(r.rental_id) as rental_count
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY rental_count DESC;
```

### Geographic Analysis
```sql
-- Customer distribution by country
SELECT country.country, COUNT(c.customer_id) as customer_count
FROM customer c
JOIN address a ON c.address_id = a.address_id
JOIN city ON a.city_id = city.city_id
JOIN country ON city.country_id = country.country_id
GROUP BY country.country_id, country.country
ORDER BY customer_count DESC;
```

## Query Optimization Tips

### Use Proper Joins
- Always join through inventory when connecting films to rentals
- Use the address->city->country chain for geographic queries
- Join through film_actor for actor-film relationships

### Date Filtering
- Use rental_date for rental period analysis
- Use return_date for analyzing return patterns (note: some rentals may not have return_date yet)
- Use payment_date for revenue analysis by time period

### Performance Considerations
- film_id, customer_id, rental_id are primary keys - use for efficient lookups
- inventory_id connects films to specific store copies
- Use LIMIT for large result sets

## Sample Business Questions and Approaches

### "What are our most popular films?"
- Define "popular" as rental count or revenue
- Join film -> inventory -> rental (for count) or -> payment (for revenue)
- Group by film and order by metric

### "Which customers generate the most revenue?"
- Join customer -> rental -> payment
- Sum payment amounts by customer
- Consider time periods for analysis

### "What's our geographic reach?"
- Join customer -> address -> city -> country
- Count customers or rentals by geographic level
- Consider store distribution as well

### "Which actors appear in the most films?"
- Join actor -> film_actor -> film
- Count distinct films per actor
- Consider both lead and supporting roles

### "What's our inventory utilization?"
- Compare total inventory to rental frequency
- Identify slow-moving vs. high-demand titles
- Consider store-specific inventory patterns

## Data Quality Notes

### Date Handling
- Some return_dates may be NULL for current rentals
- All dates are stored as DATETIME/TIMESTAMP
- Consider time zone handling if applicable

### String Data
- Actor and customer names are stored separately (first_name, last_name)
- Film titles may contain special characters
- Email addresses follow standard format validation

### Numeric Precision
- Payment amounts use DECIMAL for precise financial calculations
- Film length is stored in minutes
- Rental rates may have fractional cents

## Integration with SQLBot

When users ask about:
- "Popular movies" → Query rental counts or revenue
- "Customer analysis" → Focus on rental patterns and payment history  
- "Revenue" → Always use payment table, not estimated from rentals
- "Geographic data" → Use the address->city->country relationship chain
- "Actor information" → Remember the many-to-many relationship with films
- "Store performance" → Compare metrics across store_id values
- "Inventory" → Consider both total copies and availability status