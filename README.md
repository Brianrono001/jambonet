# jambonet
starter_project/
├── backend/
│   ... (unchanged)
├── frontend/
│   ├── Dockerfile
│   │   ```Dockerfile
│   │   FROM node:18-alpine
│   │   WORKDIR /app
│   │   COPY package*.json ./
│   │   RUN npm install
│   │   COPY . .
│   │   EXPOSE 3000
│   │   CMD ["npm", "start"]
│   │   ```
│   ... (rest unchanged)
├── docker-compose.yml
│   ```yaml
│   version: '3.8'
│   
│   services:
│     backend:
│       build: ./backend
│       ports:
│         - "8000:8000"
│       env_file:
│         - ./backend/.env.example
│       depends_on:
│         - db
│       restart: always
│
│     frontend:
│       build:
│         context: ./frontend
│         dockerfile: Dockerfile
│       ports:
│         - "3000:3000"
│       environment:
│         - REACT_APP_API_BASE=http://localhost:8000
│       stdin_open: true
│       tty: true
│
│     db:
│       image: postgres:15
│       restart: always
│       environment:
│         POSTGRES_DB: jambonet
│         POSTGRES_USER: youruser
│         POSTGRES_PASSWORD: yourpassword
│       ports:
│         - "5432:5432"
│       volumes:
│         - pgdata:/var/lib/postgresql/data
│       healthcheck:
│         test: ["CMD-SHELL", "pg_isready -U youruser"]
│         interval: 5s
│         timeout: 5s
│         retries: 5
│
│   volumes:
│     pgdata:
│   ```
├── Makefile
│   ```makefile
│   up:
│   	docker-compose up --build
│   
│   down:
│   	docker-compose down
│   
│   logs:
│   	docker-compose logs -f
│   
│   restart:
│   	docker-compose down && docker-compose up --build
│   
│   seed:
│   	docker exec -i $$(docker-compose ps -q db) psql -U youruser -d jambonet < seed.sql
│   ```
├── seed.sql
│   ```sql
│   INSERT INTO sales_records (station_id, sales_date, sales_amount)
│   VALUES
│     ('ST001', '2025-06-01', 100.0),
│     ('ST001', '2025-06-02', 110.0),
│     ('ST002', '2025-06-01', 90.0),
│     ('ST002', '2025-06-02', 95.0),
│     ('ST003', '2025-06-01', 120.0);
│   ```
├── .github/workflows/deploy.yml
│   ```yaml
│   name: Deploy Backend
│
│   on:
│     push:
│       branches:
│         - main
│
│   jobs:
│     build-and-deploy:
│       runs-on: ubuntu-latest
│       steps:
│         - uses: actions/checkout@v3
│
│         - name: Set up Python
│           uses: actions/setup-python@v4
│           with:
│             python-version: '3.11'
│
│         - name: Install dependencies
│           run: |
│             cd backend
│             pip install -r requirements.txt
│
│         - name: Run Lint
│           run: |
│             pip install flake8
│             flake8 backend
│
│         # Add deployment step here (e.g. Render CLI or SSH)
│         # Example:
│         # - name: Deploy to Render
│         #   run: render deploy service-id
│   ```
