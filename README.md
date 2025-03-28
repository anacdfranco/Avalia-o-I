<project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.seuprojeto</groupId>
  <artifactId>veiculos-api</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>veiculos-api</name>
  <description>Projeto para operacionalização da classe Veículo</description>

  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.0.0</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <java.version>17</java.version>
  </properties>

  <dependencies>
    <!-- Spring Boot Starter Web -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- Spring Boot Starter Data JPA -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- H2 Database -->
    <dependency>
      <groupId>com.h2database</groupId>
      <artifactId>h2</artifactId>
      <scope>runtime</scope>
    </dependency>
    
    <!-- PostgreSQL Driver -->
    <dependency>
      <groupId>org.postgresql</groupId>
      <artifactId>postgresql</artifactId>
      <scope>runtime</scope>
    </dependency>
    
    <!-- Validação Bean -->
    <dependency>
      <groupId>jakarta.validation</groupId>
      <artifactId>jakarta.validation-api</artifactId>
    </dependency>
    
    <!-- Testes (se necessário) -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <!-- Plugin do Spring Boot -->
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>

</project>

package com.seuprojeto;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class VeiculosApiApplication {
    public static void main(String[] args) {
        SpringApplication.run(VeiculosApiApplication.class, args);
    }
}

package com.seuprojeto.model;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.Pattern;

@Entity
@Table(name = "veiculo")
public class Veiculo {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "Marca é obrigatória")
    private String marca;

    @NotBlank(message = "Modelo é obrigatório")
    private String modelo;

    @NotBlank(message = "Placa é obrigatória")
    private String placa;

    @NotBlank(message = "CPF é obrigatório")
    @Pattern(regexp = "\\d{11}", message = "CPF deve conter 11 dígitos")
    @Column(name = "cpf_proprietario")
    private String cpfProprietario;

    // Getters e Setters

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getMarca() {
        return marca;
    }

    public void setMarca(String marca) {
        this.marca = marca;
    }

    public String getModelo() {
        return modelo;
    }

    public void setModelo(String modelo) {
        this.modelo = modelo;
    }

    public String getPlaca() {
        return placa;
    }

    public void setPlaca(String placa) {
        this.placa = placa;
    }

    public String getCpfProprietario() {
        return cpfProprietario;
    }

    public void setCpfProprietario(String cpfProprietario) {
        this.cpfProprietario = cpfProprietario;
    }
}

package com.seuprojeto.repository;

import com.seuprojeto.model.Veiculo;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;

public interface VeiculoRepository extends JpaRepository<Veiculo, Long> {
    List<Veiculo> findByCpfProprietario(String cpfProprietario);
}

package com.seuprojeto.service;

import com.seuprojeto.model.Veiculo;
import com.seuprojeto.repository.VeiculoRepository;
import com.seuprojeto.util.CpfValidator;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class VeiculoService {

    @Autowired
    private VeiculoRepository repository;

    public List<Veiculo> findAll() {
        return repository.findAll();
    }

    public Optional<Veiculo> findById(Long id) {
        return repository.findById(id);
    }

    public List<Veiculo> findByCpf(String cpf) {
        return repository.findByCpfProprietario(cpf);
    }

    public Veiculo save(Veiculo veiculo) {
        if (!CpfValidator.isValidCPF(veiculo.getCpfProprietario())) {
            throw new IllegalArgumentException("CPF inválido.");
        }
        return repository.save(veiculo);
    }

    public Veiculo update(Long id, Veiculo veiculoAtualizado) {
        Optional<Veiculo> veiculoExistente = repository.findById(id);
        if (veiculoExistente.isPresent()) {
            // Validação do CPF também na atualização
            if (!CpfValidator.isValidCPF(veiculoAtualizado.getCpfProprietario())) {
                throw new IllegalArgumentException("CPF inválido.");
            }
            veiculoAtualizado.setId(id);
            return repository.save(veiculoAtualizado);
        }
        return null;
    }

    public boolean delete(Long id) {
        if (repository.existsById(id)) {
            repository.deleteById(id);
            return true;
        }
        return false;
    }
}

package com.seuprojeto.controller;

import com.seuprojeto.model.Veiculo;
import com.seuprojeto.service.VeiculoService;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/veiculos")
public class VeiculoController {

    @Autowired
    private VeiculoService service;

    // Endpoint para listar todos os veículos
    @GetMapping
    public List<Veiculo> listarTodos() {
        return service.findAll();
    }

    // Endpoint para buscar veículo por ID
    @GetMapping("/{id}")
    public ResponseEntity<Veiculo> buscarPorId(@PathVariable Long id) {
        return service.findById(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    // Endpoint para buscar veículo(s) pelo CPF do proprietário
    @GetMapping("/cpf/{cpf}")
    public List<Veiculo> buscarPorCpf(@PathVariable String cpf) {
        return service.findByCpf(cpf);
    }

    // Endpoint para inclusão de veículo (POST)
    @PostMapping
    public ResponseEntity<Veiculo> criar(@Valid @RequestBody Veiculo veiculo) {
        return ResponseEntity.ok(service.save(veiculo));
    }

    // Endpoint para atualização de veículo (PUT)
    @PutMapping("/{id}")
    public ResponseEntity<Veiculo> atualizar(@PathVariable Long id, @Valid @RequestBody Veiculo veiculo) {
        Veiculo atualizado = service.update(id, veiculo);
        if (atualizado != null) {
            return ResponseEntity.ok(atualizado);
        }
        return ResponseEntity.notFound().build();
    }

    // Endpoint para exclusão de veículo (DELETE)
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deletar(@PathVariable Long id) {
        if (service.delete(id)) {
            return ResponseEntity.noContent().build();
        }
        return ResponseEntity.notFound().build();
    }
}

package com.seuprojeto.util;

public class CpfValidator {

    public static boolean isValidCPF(String cpf) {
        if (cpf == null || !cpf.matches("\\d{11}") || cpf.chars().distinct().count() == 1)
            return false;

        int[] peso1 = {10, 9, 8, 7, 6, 5, 4, 3, 2};
        int[] peso2 = {11, 10, 9, 8, 7, 6, 5, 4, 3, 2};

        try {
            int soma = 0;
            for (int i = 0; i < 9; i++) {
                soma += (cpf.charAt(i) - '0') * peso1[i];
            }
            int dig1 = 11 - (soma % 11);
            dig1 = (dig1 > 9) ? 0 : dig1;

            soma = 0;
            for (int i = 0; i < 10; i++) {
                soma += (cpf.charAt(i) - '0') * peso2[i];
            }
            int dig2 = 11 - (soma % 11);
            dig2 = (dig2 > 9) ? 0 : dig2;

            return dig1 == (cpf.charAt(9) - '0') && dig2 == (cpf.charAt(10) - '0');
        } catch (Exception e) {
            return false;
        }
    }
}

spring.datasource.url=jdbc:h2:mem:veiculosdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
spring.jpa.hibernate.ddl-auto=update

spring.datasource.url=jdbc:postgresql://localhost:5432/veiculosdb
spring.datasource.username=postgres
spring.datasource.password=123456
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

INSERT INTO veiculo (marca, modelo, placa, cpf_proprietario) VALUES ('Fiat', 'Uno', 'ABC1234', '12345678901');
INSERT INTO veiculo (marca, modelo, placa, cpf_proprietario) VALUES ('Chevrolet', 'Onix', 'XYZ9876', '98765432100');


