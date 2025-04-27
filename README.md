from http.server import BaseHTTPRequestHandler, HTTPServer
import http.client
import json
import urllib.parse

class WeatherServer(BaseHTTPRequestHandler):
    API_KEY = "2f08bae31a098a6825a7afce594e2f39"  # Replace with your actual API key
    
    def do_GET(self):
        if self.path == '/':
            self.send_response(200)
            self.send_header('Content-type', 'text/html')
            self.end_headers()
            
            # HTML with embedded CSS
            html = """
            <!DOCTYPE html>
            <html lang="en">
            <head>
                <meta charset="UTF-8">
                <meta name="viewport" content="width=device-width, initial-scale=1.0">
                <title>Weather App</title>
                <style>
                    body {
                        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
                        background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
                        margin: 0;
                        padding: 0;
                        display: flex;
                        justify-content: center;
                        align-items: center;
                        min-height: 100vh;
                        color: #333;
                    }
                    
                    .container {
                        background-color: white;
                        border-radius: 15px;
                        box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
                        width: 90%;
                        max-width: 500px;
                        padding: 30px;
                        text-align: center;
                    }
                    
                    h1 {
                        color: #4a6fa5;
                        margin-bottom: 30px;
                    }
                    
                    .search-box {
                        display: flex;
                        margin-bottom: 20px;
                    }
                    
                    input[type="text"] {
                        flex: 1;
                        padding: 12px 15px;
                        border: 2px solid #ddd;
                        border-radius: 25px 0 0 25px;
                        font-size: 16px;
                        outline: none;
                        transition: border 0.3s;
                    }
                    
                    input[type="text"]:focus {
                        border-color: #4a6fa5;
                    }
                    
                    button {
                        background-color: #4a6fa5;
                        color: white;
                        border: none;
                        padding: 12px 20px;
                        border-radius: 0 25px 25px 0;
                        cursor: pointer;
                        font-size: 16px;
                        transition: background-color 0.3s;
                    }
                    
                    button:hover {
                        background-color: #3a5a80;
                    }
                    
                    .weather-info {
                        margin-top: 20px;
                        padding: 20px;
                        background-color: #f8f9fa;
                        border-radius: 10px;
                        display: none;
                    }
                    
                    .weather-info h2 {
                        color: #4a6fa5;
                        margin-top: 0;
                    }
                    
                    .weather-details {
                        display: flex;
                        justify-content: space-around;
                        flex-wrap: wrap;
                        margin-top: 20px;
                    }
                    
                    .weather-item {
                        margin: 10px;
                        padding: 15px;
                        background-color: white;
                        border-radius: 10px;
                        box-shadow: 0 3px 10px rgba(0, 0, 0, 0.05);
                        min-width: 100px;
                    }
                    
                    .weather-item .value {
                        font-size: 24px;
                        font-weight: bold;
                        color: #4a6fa5;
                        margin: 5px 0;
                    }
                    
                    .weather-item .label {
                        font-size: 14px;
                        color: #666;
                    }
                    
                    .error {
                        color: #e74c3c;
                        margin-top: 20px;
                        padding: 10px;
                        background-color: #fde8e8;
                        border-radius: 5px;
                        display: none;
                    }
                    
                    .weather-icon {
                        font-size: 50px;
                        margin: 20px 0;
                    }
                </style>
            </head>
            <body>
                <div class="container">
                    <h1>Weather Forecast</h1>
                    
                    <div class="search-box">
                        <input type="text" id="city" placeholder="Enter city name">
                        <button onclick="getWeather()">Search</button>
                    </div>
                    
                    <div id="error" class="error"></div>
                    
                    <div id="weather" class="weather-info">
                        <h2 id="city-name"></h2>
                        <div class="weather-icon" id="weather-icon">☀️</div>
                        <p id="weather-desc"></p>
                        
                        <div class="weather-details">
                            <div class="weather-item">
                                <div class="value" id="temp"></div>
                                <div class="label">Temperature</div>
                            </div>
                            <div class="weather-item">
                                <div class="value" id="humidity"></div>
                                <div class="label">Humidity</div>
                            </div>
                            <div class="weather-item">
                                <div class="value" id="pressure"></div>
                                <div class="label">Pressure</div>
                            </div>
                        </div>
                    </div>
                </div>
                
                <script>
                    function getWeather() {
                        const city = document.getElementById('city').value.trim();
                        if (!city) {
                            showError('Please enter a city name');
                            return;
                        }
                        
                        fetch(`/weather?city=${encodeURIComponent(city)}`)
                            .then(response => {
                                if (!response.ok) {
                                    throw new Error('Network response was not ok');
                                }
                                return response.json();
                            })
                            .then(data => {
                                if (data.error) {
                                    showError(data.error);
                                } else {
                                    displayWeather(data);
                                }
                            })
                            .catch(error => {
                                showError('Failed to fetch weather data. Please try again.');
                                console.error('Error:', error);
                            });
                    }
                    
                    function displayWeather(data) {
                        document.getElementById('error').style.display = 'none';
                        
                        const weatherDiv = document.getElementById('weather');
                        weatherDiv.style.display = 'block';
                        
                        document.getElementById('city-name').textContent = `Weather in ${data.city}`;
                        document.getElementById('temp').textContent = `${data.temperature}°C`;
                        document.getElementById('humidity').textContent = `${data.humidity}%`;
                        document.getElementById('pressure').textContent = `${data.pressure} hPa`;
                        document.getElementById('weather-desc').textContent = data.description;
                        
                        // Set appropriate weather icon
                        const icon = document.getElementById('weather-icon');
                        const desc = data.description.toLowerCase();
                        if (desc.includes('rain')) {
                            icon.textContent = '🌧️';
                        } else if (desc.includes('cloud')) {
                            icon.textContent = '☁️';
                        } else if (desc.includes('sun') || desc.includes('clear')) {
                            icon.textContent = '☀️';
                        } else if (desc.includes('snow')) {
                            icon.textContent = '❄️';
                        } else if (desc.includes('thunder') || desc.includes('storm')) {
                            icon.textContent = '⛈️';
                        } else {
                            icon.textContent = '🌤️';
                        }
                    }
                    
                    function showError(message) {
                        document.getElementById('weather').style.display = 'none';
                        
                        const errorDiv = document.getElementById('error');
                        errorDiv.textContent = message;
                        errorDiv.style.display = 'block';
                    }
                </script>
            </body>
            </html>
            """
            self.wfile.write(html.encode('utf-8'))
            
        elif self.path.startswith('/weather?'):
            query = urllib.parse.parse_qs(urllib.parse.urlparse(self.path).query)
            city = query.get('city', [''])[0]
            
            if not city:
                self.send_response(400)
                self.send_header('Content-type', 'application/json')
                self.end_headers()
                self.wfile.write(json.dumps({'error': 'City name is required'}).encode('utf-8'))
                return
                
            weather_data = self.get_weather_data(city)
            
            self.send_response(200)
            self.send_header('Content-type', 'application/json')
            self.end_headers()
            self.wfile.write(json.dumps(weather_data).encode('utf-8'))
            
        else:
            self.send_response(404)
            self.send_header('Content-type', 'text/html')
            self.end_headers()
            self.wfile.write(b'404 Not Found')

    def get_weather_data(self, city):
        try:
            conn = http.client.HTTPSConnection("api.openweathermap.org")
            url = f"/data/2.5/weather?q={urllib.parse.quote(city)}&appid={self.API_KEY}&units=metric"
            conn.request("GET", url)
            res = conn.getresponse()
            data = res.read().decode('utf-8')
            
            if res.status == 200:
                weather_data = json.loads(data)
                main = weather_data['main']
                weather = weather_data['weather'][0]
                
                return {
                    'city': city.capitalize(),
                    'temperature': main['temp'],
                    'pressure': main['pressure'],
                    'humidity': main['humidity'],
                    'description': weather['description'].capitalize()
                }
            else:
                error_data = json.loads(data)
                return {'error': error_data.get('message', f"Unable to fetch weather data for {city}")}
                
        except Exception as e:
            return {'error': f"Error fetching weather data: {str(e)}"}
        finally:
            conn.close()

def run():
    port = 8000
    server_address = ('', port)
    httpd = HTTPServer(server_address, WeatherServer)
    print(f'Starting server on http://localhost:{port}...')
    print('Press Ctrl+C to stop the server')
    try:
        httpd.serve_forever()
    except KeyboardInterrupt:
        print("\nServer stopped")
        httpd.server_close()

if __name__ == '__main__':
    run()
