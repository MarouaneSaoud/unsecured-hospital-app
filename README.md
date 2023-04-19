# Compte Rendu📫
# Unsecured hospital app
Application de gestion des Patients 
## Author

- [Saoud Marouane ](https://www.github.com/MarouaneSaoud)

## Plan
- La logique du JPA 
- Le contexte generale de l'application
- System d'authentification
- Liste des Patients
- Ajouter Patient

## La logique du JPA
La logique du JPA, combinée avec la notion d'application fermée à la modification et ouverte à l'extension, encourage la modularité et la maintenabilité du code. En utilisant les entités et les repositories comme parties fermées de l'application pour gérer la persistance des données, et en ajoutant de nouvelles entités et repositories pour ajouter de nouvelles fonctionnalités sans modifier le code existant, on peut concevoir une application extensible et réutilisable.


## Le contexte generale de l'application
L'idée de cette application est de créer un système de gestion de patients utilisant Spring Boot et JPA pour la persistance des données. L'application permet aux administrateurs de gérer les informations des patients, tandis que les utilisateurs simples peuvent seulement visualiser la liste des patients. Les fonctionnalités principales de l'application incluent l'ajout, la suppression, et la modification des patients, ainsi que l'affichage des patients dans un tableau.

Pour faciliter la visualisation des patients, l'application fournira une fonctionnalité d'affichage paginé des patients dans un tableau. Cela permettra aux utilisateurs de naviguer à travers la liste des patients de manière conviviale et de rechercher des patients spécifiques en utilisant des filtres tels que le nom, le prénom, ou d'autres critères.

L'application permettra aux administrateurs de se connecter à un compte sécurisé avec des autorisations spécifiques pour effectuer des opérations de gestion des patients. Une fois connecté, l'administrateur pourra ajouter de nouveaux patients en fournissant des informations. Les administrateurs auront également la possibilité de modifier les informations des patients existants.

En outre, l'application offrira la possibilité aux administrateurs de supprimer les patients du système si nécessaire. La suppression des patients sera gérée de manière sécurisée, en s'assurant que seuls les administrateurs autorisés peuvent effectuer cette opération.

Il est important de noter que seuls les administrateurs auront la possibilité d'effectuer des opérations de modification ou de suppression des patients, tandis que les utilisateurs simples pourront uniquement visualiser la liste des patients sans pouvoir effectuer de modifications.

En suivant les principes de l'application fermée à la modification et ouverte à l'extension, l'application sera conçue de manière modulaire et extensible, permettant l'ajout de nouvelles fonctionnalités sans avoir à modifier le code existant. Elle sera également sécurisée, en utilisant l'authentification et l'autorisation pour garantir que seuls les utilisateurs autorisés peuvent effectuer des opérations de gestion des patients.


<details>

<summary> Quelque dependance utiliser au cours du developpemnt</summary>

#### Thymeleaf
```xml
   <dependency>
            <groupId>org.thymeleaf.extras</groupId>
            <artifactId>thymeleaf-extras-springsecurity6</artifactId>
            <version>3.1.0.M1</version>
   </dependency>
```
#### Spring boot security
 ```xml
   <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
 ```

#### lombok
 ```xml
  <dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
 ```
#### mysql
 ```xml
 <dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
 ```

</details>

## System d'authentification


``Avec Spring Security, nous avons mis en place un système d'authentification robuste qui assure que seuls les utilisateurs autorisés peuvent accéder à notre application. De plus, nous utilisons des fonctionnalités avancées telles que la gestion des rôles et des autorisations pour garantir que les utilisateurs ont uniquement accès aux ressources qui leur sont autorisées. ``

#### Page d'authentification
![App Screenshot](/image/Auth.png)

###### Class SecurityConfig

 ```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true)
public class SecurityConfig {
    @Autowired
    private PasswordEncoder passwordEncoder;

    @Bean
    public InMemoryUserDetailsManager inMemoryUserDetailsManager(){
        return new InMemoryUserDetailsManager(
                User.withUsername("user1").password(passwordEncoder.encode("1234")).roles("USER").build(),
                User.withUsername("user2").password(passwordEncoder.encode("1234")).roles("USER").build(),
                User.withUsername("admin").password(passwordEncoder.encode("1234")).roles("USER","ADMIN").build()
        );

    }
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity httpSecurity)throws Exception{
        httpSecurity.formLogin().loginPage("/login")
                .defaultSuccessUrl("/")
                .permitAll();
        //httpSecurity.rememberMe();
        httpSecurity.authorizeHttpRequests().requestMatchers("/webjars/**","/h2-console/**").permitAll();
        // httpSecurity.authorizeHttpRequests().requestMatchers("/user/**").hasRole("USER");
        //httpSecurity.authorizeHttpRequests().requestMatchers("/admin/**").hasRole("ADMIN");
        httpSecurity.authorizeHttpRequests().anyRequest().authenticated();
        httpSecurity.exceptionHandling().accessDeniedPage("/notAuthorized");
        return httpSecurity.build();

    }
}

 ```

## Liste des Patients

`` Visualiser la liste des patients: Cette fonctionnalité permet aux utilisateurs simples de visualiser la liste des patients existants dans le tableau, sans pouvoir supprimer ou modifier les patients. Les utilisateurs simples peuvent simplement parcourir les pages du tableau pour consulter les informations des patients sans avoir la possibilité de les modifier ou de les supprimer. ``

![App Screenshot](/image/ListeDesPatient.png)

#### Affichage ses patients dans un tableau 
`` Cette fonctionnalité permet d'afficher et chercher les patients existants dans un tableau, avec une pagination pour limiter le nombre de patients affichés à la fois. Les utilisateurs peuvent naviguer à travers les différentes pages pour visualiser les patients de manière organisée et conviviale. Cela peut inclure l'affichage des informations importantes sur chaque patient, telles que le nom, date naissance, etc. ``


###### Controlleur d'affichage des Patient
```java
@GetMapping("/user/index")
public String index(Model model,
@RequestParam(name = "page",defaultValue = "0") int page,
@RequestParam(name = "size",defaultValue = "5") int size,
@RequestParam(name = "keyword",defaultValue = "") String kw
        ){
        Page<Patient> pagePatients = patientRepository.findByNomContains(kw, PageRequest.of(page,size));
        model.addAttribute("listPatients",pagePatients.getContent());
        model.addAttribute("pages",new int[pagePatients.getTotalPages()]);
        model.addAttribute("currentPage",page);
        model.addAttribute("keyword",kw);
        return "patients";
        }

 ```
#### Supprimer un patient (réservé aux administrateurs)
``Cette fonctionnalité permet aux administrateurs de supprimer un patient existant de la base de données de l'application. Les administrateurs peuvent sélectionner un patient à supprimer dans le tableau affiché et confirmer leur choix. Une fois confirmée, l'application supprime le patient sélectionné de la base de données. ``
###### Controlleur de Suppression d'un Patient
```java
   @GetMapping("/admin/deletePatient")
    @PreAuthorize("hasRole('ROLE_ADMIN')")
    public String deletePatient(@RequestParam(name = "id") Long id, String keyword, int page){
        patientRepository.deleteById(id);
        return "redirect:/user/index?page="+page+"&keyword="+keyword;
    }
```
#### Modifier un patient (réservé aux administrateurs)
`` Cette fonctionnalité permet aux administrateurs de modifier les informations d'un patient existant dans la base de données. Les administrateurs peuvent sélectionner un patient dans le tableau affiché, puis mettre à jour les informations nécessaires. Une fois les modifications effectuées, l'application enregistre les nouvelles  ``

![App Screenshot](/image/UpdatePatient.png)
```java
    @GetMapping("/admin/editPatient")
    @PreAuthorize("hasRole('ROLE_ADMIN')")
    public String editPatient(@RequestParam(name = "id") Long id, Model model){
            Patient patient=patientRepository.findById(id).get();
            model.addAttribute("patient",patient);
            return "editPatient";
            }
```
## Ajouter Patient 

``Ajouter un patient : Cette fonctionnalité permet aux administrateurs d'ajouter de nouveaux patients à la base de données de l'application. Elle peut inclure la saisie d'informations telles que le nom, la date de naissance, le score, est ce qu'il est malade ou non ,  Les informations fournies par l'administrateur sont ensuite enregistrées dans la base de données Mysql pour créer un nouveau patient. ``

#### Page d'ajout du patient (réservé aux administrateurs)
![App Screenshot](/image/FormPatient.png)

###### Controlleur d'ajout du patient
```java
@GetMapping("/admin/formPatient")
@PreAuthorize("hasRole('ROLE_ADMIN')")
public String formPatient(Model model ){
        model.addAttribute("patient",new Patient());
        return "formPatient";
        }
        
@PostMapping("/admin/savePatient")
@PreAuthorize("hasRole('ROLE_ADMIN')")
public String savePatient(@Valid Patient patient, BindingResult bindingResult){
        if (bindingResult.hasErrors()) return "formPatient";
        patientRepository.save(patient);
        return "formPatient";
        }
 ```


