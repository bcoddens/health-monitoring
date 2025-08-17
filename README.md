# Health Monitoring System

A comprehensive health monitoring system integrating **Home Assistant**, **Node-RED**, **Garmin Connect**, **Intervals.icu**, and **Xert** for tracking physical, mental, and emotional health metrics with advanced MCP server integration.

## 🏥 Features

- **Comprehensive Health Tracking**: Physical health, recovery, sleep, mental health, and environmental factors
- **Cycling-Specific Metrics**: Power analysis, training zones, HRV-based readiness, Alan Couzens methodology
- **Advanced Analytics**: Xert integration for dynamic fitness signatures and training recommendations
- **MCP Server Integration**: Serena, Context7, and Home Assistant MCP servers for enhanced automation
- **Docker Environment**: Containerized deployment with health monitoring utilities

## 🚀 Quick Start

### Prerequisites
- Docker and Docker Compose
- Node.js 18+ (for MCP servers)
- Home Assistant long-lived access token
- Garmin Connect, Intervals.icu, and/or Xert accounts (optional)

### Setup

1. **Clone and configure**:
   ```bash
   git clone <repository-url>
   cd health-monitoring
   cp .env.example .env
   # Edit .env with your credentials
   ```

2. **Start services**:
   ```bash
   ./scripts/health-docker-utils.sh start
   ```

3. **Check status**:
   ```bash
   ./scripts/health-docker-utils.sh status
   ```

## 📊 Services

- **Home Assistant** (port 8123) - Central health monitoring hub
- **Node-RED** (port 1880) - Automation flows and data processing
- **Mosquitto MQTT** (ports 1883/9001) - Sensor data messaging
- **MCP Services** - Advanced AI integration and analysis

## 🛠 Management Commands

```bash
# Start all services
./scripts/health-docker-utils.sh start

# Stop all services  
./scripts/health-docker-utils.sh stop

# Show service status
./scripts/health-docker-utils.sh status

# Run health checks
./scripts/health-docker-utils.sh health

# View logs
./scripts/health-docker-utils.sh logs [service] [lines]

# Restart specific service
./scripts/health-docker-utils.sh restart [service]

# Backup configuration
./scripts/health-docker-utils.sh backup
```

## 🔧 MCP Servers

### Available MCP Servers
- **Serena MCP**: Semantic code analysis and pattern search
- **Context7 MCP**: Up-to-date library documentation and API patterns
- **Home Assistant MCP**: Direct Home Assistant integration and automation

### Testing MCP Servers
```bash
# Test Serena MCP
node scripts/serena-mcp-bridge.js test

# Test Context7 MCP
echo '{"jsonrpc": "2.0", "id": 1, "method": "tools/list"}' | node scripts/context7-mcp-server.js

# Test Home Assistant MCP
echo '{"jsonrpc": "2.0", "id": 1, "method": "tools/list"}' | node scripts/home-assistant-mcp-server.js
```

## 📈 Health Metrics Tracked

### Physical Health
- Heart rate (resting, recovery, training)
- VO2max and cardio fitness
- Blood pressure and body composition
- Cycling-specific metrics (cadence, power, lactate threshold zones)

### Recovery & Sleep
- Sleep duration and quality (Garmin integration)
- HRV and recovery metrics
- Sleep environment monitoring
- HRV-based readiness assessment (Alan Couzens methodology)

### Mental & Emotional Health
- Stress levels and mood tracking
- Meditation and mindfulness practice
- Social connections and work-life balance
- Sense of purpose and life satisfaction

### Environmental Health
- Air quality and temperature monitoring
- Light exposure and noise levels
- Water quality tracking

## 🎯 Advanced Features

### Alan Couzens Training Methodology
- HRV-based training load management
- Nutritional periodization based on training volume
- Pyramidal training distribution tracking (Norwegian model)
- Adaptive training recommendations

### Xert Integration
- Dynamic fitness signature tracking
- Real-time training status and recommendations
- Breakthrough detection and FTP updates
- Advanced power analysis

## 📋 Configuration

See [CLAUDE.md](./CLAUDE.md) for comprehensive configuration details, including:
- Home Assistant setup and integrations
- Node-RED flow configurations
- Garmin Connect, Intervals.icu, and Xert API setup
- Dashboard configurations and automations
- MCP server integration details

## 🔒 Security & Privacy

- All health data stored locally in Home Assistant
- Environment variable based credential management
- Docker container isolation
- Optional encryption for sensitive data

## 🐛 Troubleshooting

### Common Issues
1. **Services won't start**: Check Docker daemon and run `./scripts/health-docker-utils.sh health`
2. **MCP servers failing**: Verify Node.js dependencies with `npm install`
3. **Home Assistant integration issues**: Check credentials in `.env` file
4. **Permission errors**: Ensure proper file permissions for config directories

### Debug Commands
```bash
# Check Docker environment
./scripts/health-docker-utils.sh health

# View service logs
./scripts/health-docker-utils.sh logs homeassistant 50

# Test individual MCP servers
node scripts/serena-mcp-bridge.js test
```

## 📄 License

This project is licensed under the MIT License.

## 🤝 Contributing

Contributions welcome! Please read the contribution guidelines and ensure all health monitoring best practices are followed.

---

**Note**: This system is designed for personal health monitoring and should not replace professional medical advice. Always consult healthcare professionals for medical decisions.