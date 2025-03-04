import pandas as pd
import requests
import time

def geocode_address(address):
    """
    Returns (lat, lon) or (None, None) via Nominatim.
    """
    url = "https://nominatim.openstreetmap.org/search"
    headers = { "User-Agent": "MyGeocoderApp/1.0 (contact: your_email@example.com)" }
    params = {
        'q': address,
        'format': 'json',
        'addressdetails': 1,
        'limit': 1
    }
    response = requests.get(url, params=params, headers=headers)
    data = response.json()
    if data:
        lat = float(data[0]['lat'])
        lon = float(data[0]['lon'])
        return lat, lon
    else:
        return None, None

def classify_address_by_socrata(lat, lon):
    """
    Queries the City of Austin Socrata endpoint (vnwj-xmz9) to see if (lat, lon) 
    is within a polygon labeled 'FULL PURPOSE' or 'ETJ'.
    
    Returns a string like "AFP", "ETJ", or "Unknown".
    """
    base_url = "https://data.austintexas.gov/resource/vnwj-xmz9.json"
    
    point_str = f"POINT({lon} {lat})"
    where_clause = f"within_polygon(the_geom, '{point_str}')"
    
    params = {
        "$where": where_clause,
        "$limit": 1
    }
    
    try:
        response = requests.get(base_url, params=params)
        data = response.json()
        
        if not data:
            return "Unknown"
        
        row = data[0]  # first result

        jurisdiction_type = row.get("jurisdiction_type", "").upper()
        
        if "AFP" in jurisdiction_type:
            return "Austin Full Purpose"
        elif "2MILE" in jurisdiction_type:
            return "Austin 2 Mile ETJ"
        elif "5MILE" in jurisdiction_type:
            return "Austin 5 Mile ETJ"
        elif "LTD" in jurisdiction_type:
            return "Austin LTD"
        else:
            return "Unknown"
    
    except Exception as e:
        print("Error calling Socrata boundary API:", e)
        return "Unknown"
