---
source_course: "redis-fundamentals"
source_lesson: "redis-fundamentals-geospatial"
---

# Geospatial: Location Data

Redis Geospatial indices let you store latitude/longitude coordinates and perform location-based queries. Built on top of Sorted Sets, they're perfect for finding nearby locations.

## Adding Locations

### GEOADD

```redis
# Add single location (longitude, latitude, name)
GEOADD stores -122.4194 37.7749 "store:sf"
# Returns: 1

# Add multiple locations
GEOADD stores \
  -118.2437 34.0522 "store:la" \
  -73.9857 40.7484 "store:nyc" \
  -87.6298 41.8781 "store:chicago"
# Returns: 3
```

**Note**: Longitude comes before latitude (opposite of Google Maps).

## Querying Locations

### Get Position

```redis
# Get coordinates of a location
GEOPOS stores "store:sf"
# Returns: [["-122.41940140724182", "37.77490137100031"]]

# Multiple locations
GEOPOS stores "store:sf" "store:la"
```

### Calculate Distance

```redis
# Distance between two locations
GEODIST stores "store:sf" "store:la"
# Returns: "559119.4938" (meters by default)

# In different units
GEODIST stores "store:sf" "store:la" km    # Kilometers
# Returns: "559.1195"

GEODIST stores "store:sf" "store:la" mi    # Miles
# Returns: "347.3998"
```

## Finding Nearby Locations

### GEOSEARCH

```redis
# Find locations within radius of a point
GEOSEARCH stores FROMMEMBER "store:sf" BYRADIUS 600 km
# Returns: ["store:sf", "store:la"]

# Search from coordinates
GEOSEARCH stores FROMLONLAT -122.4 37.7 BYRADIUS 50 mi
# Returns locations within 50 miles

# Get with distances
GEOSEARCH stores FROMMEMBER "store:sf" BYRADIUS 1000 km WITHDIST
# Returns: [["store:sf", "0.0000"], ["store:la", "559.1195"]]

# Get with coordinates
GEOSEARCH stores FROMMEMBER "store:sf" BYRADIUS 1000 km WITHCOORD

# Limit results
GEOSEARCH stores FROMMEMBER "store:sf" BYRADIUS 1000 km COUNT 5

# Sort by distance
GEOSEARCH stores FROMMEMBER "store:sf" BYRADIUS 5000 km ASC
```

### Search by Box

```redis
# Find within rectangular area
GEOSEARCH stores FROMLONLAT -100 40 BYBOX 4000 2000 km
```

## Geohash

```redis
# Get geohash of locations (useful for clustering)
GEOHASH stores "store:sf" "store:la"
# Returns: ["9q8yyk8y9v0", "9q5ctr186m0"]
```

## Practical Example: Store Locator

```redis
# Add all store locations
GEOADD stores:coffee \
  -122.4089 37.7855 "Blue Bottle Hayes" \
  -122.4004 37.7867 "Blue Bottle Mint" \
  -122.4194 37.7749 "Starbucks Market" \
  -122.4058 37.7879 "Philz Coffee"

# User is at Union Square, find nearest 3 coffee shops
GEOSEARCH stores:coffee \
  FROMLONLAT -122.4075 37.7879 \
  BYRADIUS 1 mi \
  WITHDIST \
  COUNT 3 \
  ASC
# Returns nearest 3 shops with distances

# Store result in new key
GEOSEARCHSTORE nearby:user:1 stores:coffee \
  FROMLONLAT -122.4075 37.7879 \
  BYRADIUS 1 mi \
  COUNT 10
```

## Practical Example: Delivery Zone Check

```redis
# Add delivery zones (center points)
GEOADD delivery:zones \
  -122.4194 37.7749 "zone:downtown" \
  -122.4512 37.7654 "zone:sunset"

# Check if customer address is within delivery range
GEOADD temp:customer -122.4300 37.7700 "customer"
GEODIST delivery:zones "zone:downtown" "customer" mi
# If < 5 miles, they're in the delivery zone
DEL temp:customer
```

📖 [Geospatial Commands](https://redis.io/docs/latest/commands/?group=geo)

## Resources

- [Redis Geospatial](https://redis.io/docs/latest/develop/data-types/geospatial/) — Complete guide to Redis Geospatial indices

---

> 📘 *This lesson is part of the [In-Memory Data Store Fundamentals](https://stanza.dev/courses/redis-fundamentals) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*