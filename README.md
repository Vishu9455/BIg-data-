# BIg-data-
/* Project Structure:

- src/main/java/com/example/pickspot/
    - PickSpotApplication.java
    - controller/PickSpotController.java
    - model/Container.java
    - model/YardSlot.java
    - model/PickSpotRequest.java
    - model/PickSpotResponse.java
    - service/PickSpotService.java

- src/main/resources/
    - application.properties

- pom.xml
- README.md
*/

// pom.xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>pickspot</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.3</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>

// src/main/java/com/example/pickspot/PickSpotApplication.java
package com.example.pickspot;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class PickSpotApplication {
    public static void main(String[] args) {
        SpringApplication.run(PickSpotApplication.class, args);
    }
}

// src/main/java/com/example/pickspot/controller/PickSpotController.java
package com.example.pickspot.controller;

import com.example.pickspot.model.PickSpotRequest;
import com.example.pickspot.model.PickSpotResponse;
import com.example.pickspot.service.PickSpotService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/pickSpot")
public class PickSpotController {

    @Autowired
    private PickSpotService pickSpotService;

    @PostMapping
    public PickSpotResponse pickSpot(@RequestBody PickSpotRequest request) {
        return pickSpotService.findBestSpot(request);
    }
}

// src/main/java/com/example/pickspot/model/Container.java
package com.example.pickspot.model;

public class Container {
    private String id;
    private String size;
    private boolean needsCold;
    private int x;
    private int y;

    // Getters and Setters
}

// src/main/java/com/example/pickspot/model/YardSlot.java
package com.example.pickspot.model;

public class YardSlot {
    private int x;
    private int y;
    private String sizeCap;
    private boolean hasColdUnit;
    private boolean occupied;

    // Getters and Setters
}

// src/main/java/com/example/pickspot/model/PickSpotRequest.java
package com.example.pickspot.model;

import java.util.List;

public class PickSpotRequest {
    private Container container;
    private List<YardSlot> yardMap;

    // Getters and Setters
}

// src/main/java/com/example/pickspot/model/PickSpotResponse.java
package com.example.pickspot.model;

import java.util.Map;

public class PickSpotResponse {
    private Map<String, Object> container;
    private String error;

    // Getters and Setters
}

// src/main/java/com/example/pickspot/service/PickSpotService.java
package com.example.pickspot.service;

import com.example.pickspot.model.Container;
import com.example.pickspot.model.PickSpotRequest;
import com.example.pickspot.model.PickSpotResponse;
import com.example.pickspot.model.YardSlot;
import org.springframework.stereotype.Service;

import java.util.HashMap;
import java.util.Map;

@Service
public class PickSpotService {

    private static final int INVALID = 10_000;

    public PickSpotResponse findBestSpot(PickSpotRequest request) {
        Container container = request.getContainer();
        int minScore = Integer.MAX_VALUE;
        YardSlot bestSlot = null;

        for (YardSlot slot : request.getYardMap()) {
            int score = computeScore(container, slot);
            if (score < minScore) {
                minScore = score;
                bestSlot = slot;
            }
        }

        PickSpotResponse response = new PickSpotResponse();

        if (bestSlot == null || minScore >= INVALID) {
            response.setError("no suitable slot");
        } else {
            Map<String, Object> containerPlacement = new HashMap<>();
            containerPlacement.put("containerId", container.getId());
            containerPlacement.put("targetX", bestSlot.getX());
            containerPlacement.put("targetY", bestSlot.getY());
            response.setContainer(containerPlacement);
        }

        return response;
    }

    private int computeScore(Container container, YardSlot slot) {
        int distance = Math.abs(container.getX() - slot.getX()) + Math.abs(container.getY() - slot.getY());
        int sizePenalty = ("big".equals(container.getSize()) && "small".equals(slot.getSizeCap())) ? INVALID : 0;
        int coldPenalty = (container.isNeedsCold() && !slot.isHasColdUnit()) ? INVALID : 0;
        int occupiedPenalty = slot.isOccupied() ? INVALID : 0;

        return distance + sizePenalty + coldPenalty + occupiedPenalty;
    }
}

// src/main/resources/application.properties
# Empty (default Spring Boot settings)

// README.md
# PickSpot Service

## How to Run

```bash
mvn spring-boot:run
```

## Sample curl

```bash
curl -X POST http://localhost:8080/pickSpot \
-H "Content-Type: application/json" \
-d '{
  "container": {
    "id": "C1",
    "size": "small",
    "needsCold": false,
    "x": 1,
    "y": 1
  },
  "yardMap": [
    { "x": 1, "y": 2, "sizeCap": "small", "hasColdUnit": false, "occupied": false },
    { "x": 2, "y": 2, "sizeCap": "big", "hasColdUnit": true, "occupied": false }
  ]
}'
```

### Sample Response
```json
{
  "container": {
    "containerId": "C1",
    "targetX": 1,
    "targetY": 2
  }
}
```

---

Want me to package this into a downloadable ZIP next? 🚀 (Say: **"zip it"** )
