## Integration of a Mathematical Calulations with a Chat Completion System using LLM Function-Calling

### AIM:
To design and implement a Python function for calculating the volume of a cylinder, integrate it with a chat completion system utilizing the function-calling feature of a large language model (LLM).

## THEORY:
The system combines:
1. **Mathematical Component**: Uses the formula V = πr²h for cylinder volume calculation
2. **AI Component**: Utilizes Google's Gemini model for natural language processing
3. **Integration Layer**: Connects the AI understanding with mathematical computation

## ALGORITHM:
1. Initialize Gemini AI and configure API key
2. Accept natural language input from user
3. Process query through Gemini to extract parameters
4. Calculate cylinder volume using extracted parameters
5. Format and display results
6. Handle any errors gracefully

## PROGRAM:
```python
import math
from typing import Dict, Any
import google.generativeai as genai
import json

# Configuration
GOOGLE_API_KEY = "your-api-key-here"  
genai.configure(api_key=GOOGLE_API_KEY)

def calculate_cylinder_volume(radius: float, height: float) -> float:
    
    return math.pi * (radius ** 2) * height

def create_gemini_prompt(user_input: str) -> str:
    
    return f"""
    Extract the radius and height values from the following query about a cylinder.
    Return only a JSON object with 'radius' and 'height' as keys and their values in meters.
    
    Query: {user_input}
    
    Expected format:
    {{"radius": number, "height": number}}
    """

def parse_gemini_response(response_text: str) -> Dict[str, float]:
    """
    Parse Gemini's response to extract radius and height values.
    
    Args:
        response_text (str): Response from Gemini model
    
    Returns:
        Dict[str, float]: Extracted parameters
    """
    try:
        # Find the JSON object in the response
        start_idx = response_text.find('{')
        end_idx = response_text.rfind('}') + 1
        json_str = response_text[start_idx:end_idx]
        
        # Parse the JSON object
        params = json.loads(json_str)
        
        # Validate required keys
        if 'radius' not in params or 'height' not in params:
            raise ValueError("Missing required parameters in response")
            
        return {
            'radius': float(params['radius']),
            'height': float(params['height'])
        }
    except Exception as e:
        raise ValueError(f"Failed to parse Gemini response: {str(e)}")

def process_user_query(user_input: str) -> Dict[str, Any]:
    
    try:
        # Initialize Gemini model
        model = genai.GenerativeModel('gemini-pro')
        
        # Create and send prompt to Gemini
        prompt = create_gemini_prompt(user_input)
        response = model.generate_content(prompt)
        
        # Parse parameters from response
        parameters = parse_gemini_response(response.text)
        
        # Calculate the volume
        volume = calculate_cylinder_volume(parameters['radius'], parameters['height'])
        
        return {
            "success": True,
            "volume": volume,
            "units": "cubic meters",
            "input_parameters": parameters
        }
    
    except Exception as e:
        return {
            "success": False,
            "error": str(e)
        }

def format_volume(volume: float) -> str:
   
    if volume >= 1:
        return f"{volume:.2f} cubic meters"
    else:
        return f"{volume * 1000:.2f} cubic centimeters"

def main():
    
    print("=== Cylinder Volume Calculator with Gemini Integration ===\n")
    print("Enter 'quit' to exit\n")
    
    while True:
        user_input = input("Enter your query (e.g., 'Calculate volume of cylinder with radius 3m and height 5m'): ")
        
        if user_input.lower() == 'quit':
            break
            
        result = process_user_query(user_input)
        
        if result["success"]:
            volume_str = format_volume(result["volume"])
            print(f"\nResults:")
            print(f"- Volume: {volume_str}")
            print(f"- Dimensions used:")
            print(f"  * Radius: {result['input_parameters']['radius']} meters")
            print(f"  * Height: {result['input_parameters']['height']} meters")
        else:
            print(f"\nError: {result['error']}")
        
        print("\n" + "-" * 50 + "\n")

if __name__ == "__main__":
    main()
```

### OUTPUT:
![alt text](Image.png)

### RESULT:
