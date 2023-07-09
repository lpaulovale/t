# Inserção De Contatos
## 1. Crie o diretorio
  Abra o cmd no windows e insira os comandos.
 ````
 mkdir contato
 cd contato 
````

 
## 2. Inicialize projeto e instale as dependencias
   Insira os comandos 
````
npm init
npm install --save express body-parser sequelize sequelize-cli sqlite3 nodemon
mkdir static
````
## 3. Tela de saída
  Crie um arquivo index.html no diretorio ./static/ e insira o código.
````
      <!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Contacts</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bulma/0.7.1/css/bulma.css">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/vue/2.5.17-beta.0/vue.js"></script>
  <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css">
  <style>
    .section-container {
      max-width: 800px;
      margin-right: auto;
      margin-left: auto;
    }
  </style>
</head>
<body>
  <div id="app" class="container">
    <section class="section section-container" style="padding-top: 24px; padding-bottom: 5px;">
        <h2 class="title">Contacts</h2>
        <contact v-for="contact in contacts"
            :key="contact.name"
            :contact="contact"
            @save-contact="onAddOrUpdateContact" 
            @delete-contact="deleteContact" />
    </section>
    <section class="section section-container" style="padding-bottom: 10px;">
      <div class="box">
        <add-update-contact title="Add Contact" @save-contact="onAddOrUpdateContact" />
      </div>
    </section>
  </div>
  <script>
  const AddUpdateContact = {
    props: ['contact', 'title'],
    data () {
      return {
        id: this.contact ? this.contact.id : null,
        firstName: this.contact ? this.contact.firstName : '',
        lastName: this.contact ? this.contact.lastName : '',
        phone: this.contact ? this.contact.phone : ''
      }
    },
    methods: {
      save() {
        this.$emit('save-contact', { id: this.id, firstName: this.firstName, lastName: this.lastName, phone: this.phone })
        if (!this.id) {
          this.firstName = ''
          this.lastName = ''
          this.phone = ''
        }
      }
    },
    template: `
      <form class="form" @submit.prevent="save">
        <h3 class='subtitle'>{{ title }}</h3>
        <div class="field">
            <label>First Name</label>
            <div class="control">
              <input class="input" type="text" v-model="firstName">
            </div> 
        </div>
        <div class="field">
            <label>Last Name</label>
            <div class="control">
              <input class="input" type="text" v-model="lastName">
            </div> 
        </div>
        <div class="field">
            <label>Phone</label>
            <div class="control">
              <input class="input" type="text" v-model="phone">
            </div> 
        </div>
        <div class="field">
            <div class="control">
              <button class="button is-success">Save</button>
            </div> 
        </div>
      </form>
    `
  }

  const Contact = {
    props: ['contact'],
    components: { 'add-update-contact': AddUpdateContact },
    data () {
      return {
        showDetail: false
      }
    },
    methods: {
      onAddOrUpdateContact(contact) {
        this.$emit('save-contact', contact)
      },
      deleteContact (contact) {
        this.$emit('delete-contact', contact)
      }
    },
    template: `
      <div class="card">
        <header class="card-header">
          <p @click="showDetail = !showDetail" class="card-header-title">
            {{ contact.firstName }} {{ contact.lastName }}
          </p>
          <a class="card-header-icon" @click.stop="deleteContact(contact)">
            <span class="icon">
              <i class="fa fa-trash"></i>
            </span>
          </a>
        </header>
        <div v-show="showDetail" class="card-content">
            <add-update-contact title="Details" :contact="contact" @save-contact="onAddOrUpdateContact" />
        </div>
      </div>
    `
  }

  new Vue({
    el: '#app',
    components: { contact: Contact, 'add-update-contact': AddUpdateContact },
    data: {
      contacts: [],
      apiURL: 'http://localhost:3000/api/contacts'
    },
    methods: {
      onAddOrUpdateContact (contact) {
        if (contact.id) {
          this.updateContact(contact)
        } else {
          this.addContact(contact)
        }
      },
      addContact (contact) {
        return axios.post(this.apiURL, contact)
          .then((response) => {
            const copy = this.contacts.slice()
            copy.push(response.data)
            this.contacts = copy
          })
      },
      updateContact (contact) {
        return axios.put(`${this.apiURL}/${contact.id}`, contact)
          .then((response) => {
            const copy = this.contacts.slice()
            const idx = copy.findIndex((c) => c.id === response.data.id)
            copy[idx] = response.data
            this.contacts = copy
          })
      },
      deleteContact (contact) {
        console.log('deleting', contact)
        return axios.delete(`${this.apiURL}/${contact.id}`)
          .then((response) => {
            let copy = this.contacts.slice()
            const idx = copy.findIndex((c) => c.id === response.data.id)
            copy.splice(idx, 1)
            this.contacts = copy
          })
      }
    },
    beforeMount () {
      axios.get(this.apiURL)
        .then((response) => {
          this.contacts = response.data
        })
    }
  })

  </script>
</body>
</html>
````
## 4. Inicialize as dependencias e configure com o sequilize 
````
	 node_modules/.bin/sequelize
````
Substitua o conteudo do arquivo ./config/config.json pelo código.
````
  {
  "development": {
    "dialect": "sqlite",
    "storage": "./database.sqlite3"
  },
  "test": {
    "dialect": "sqlite",
    "storage": ":memory"
  },
  "production": {
    "dialect": "sqlite",
    "storage": "./database.sqlite3"
  }
}
````
## 5. Crie o model
````
 node_modules/.bin/sequelize model:generate --name Contact --attributes firstName:string,lastName:string,phone:string,email:string
````
## 6. Insira a tabela no sqlite
````
node_modules/.bin/sequelize db:migrate
````
##7. Verificando dependencias
````
sqlite3 database.sqlite3
.schema
````  
## 8. Crie um arquivo de migrações para o contacts
````
.headers ON
select * from SequelizeMeta;
````
## 9. Crie arquivo seed
````
node_modules/.bin/sequelize seed:generate --name seed-contact
````
Vá para ./seeders/20230709211745-seed-contact.js substitua o conteudo pelo código.
````
'use strict';

module.exports = {
  up: (queryInterface, Sequelize) => {
    /*
      Add altering commands here.
      Return a promise to correctly handle asynchronicity.

      Example:
      return queryInterface.bulkInsert('Person', [{
        name: 'John Doe',
        isBetaMember: false
      }], {});
    */
   return queryInterface.bulkInsert('Contacts', [{
      firstName: 'Snoop',
      lastName: 'Dog',
      phone: '111-222-3333',
      email: 'snoopydog@dogpound.com',
      createdAt: new Date().toDateString(),
      updatedAt: new Date().toDateString()
    }, {
      firstName: 'Scooby',
      lastName: 'Doo',
      phone: '444-555-6666',
      email: 'scooby.doo@misterymachine.com',
      createdAt: new Date().toDateString(),
      updatedAt: new Date().toDateString()
    }, {
      firstName: 'Herbie',
      lastName: 'Husker',
      phone: '402-437-0001',
      email: 'herbie.husker@unl.edu',
      createdAt: new Date().toDateString(),
      updatedAt: new Date().toDateString()
    }], {});
  },

  down: (queryInterface, Sequelize) => {
    /*
      Add reverting commands here.
      Return a promise to correctly handle asynchronicity.

      Example:
      return queryInterface.bulkDelete('Person', null, {});
    */
   return queryInterface.bulkDelete('Contacts', null, {});
  }
};
````
````
node_modules/.bin/sequelize db:seed:all
````
## 5. Crie o servidor e execute.
 Crie um arquivo ./server.js e insira o código.
````
// server.js

const express = require('express');
const bodyParser = require('body-parser');
const db = require('./models');

const app = express();

app.use(bodyParser.json());
app.use(express.static(__dirname + '/static'));

app.get('/api/contacts', (req, res) => {
  return db.Contact.findAll()
    .then((contacts) => res.send(contacts))
    .catch((err) => {
      console.log('There was an error querying contacts', JSON.stringify(err))
      return res.send(err)
    });
});

app.post('/api/contacts', (req, res) => {
  const { firstName, lastName, phone } = req.body
  return db.Contact.create({ firstName, lastName, phone })
    .then((contact) => res.send(contact))
    .catch((err) => {
      console.log('***There was an error creating a contact', JSON.stringify(contact))
      return res.status(400).send(err)
    })
});

app.delete('/api/contacts/:id', (req, res) => {
  const id = parseInt(req.params.id)
  return db.Contact.findById(id)
    .then((contact) => contact.destroy({ force: true }))
    .then(() => res.send({ id }))
    .catch((err) => {
      console.log('***Error deleting contact', JSON.stringify(err))
      res.status(400).send(err)
    })
});

app.put('/api/contacts/:id', (req, res) => {
  const id = parseInt(req.params.id)
  return db.Contact.findById(id)
  .then((contact) => {
    const { firstName, lastName, phone } = req.body
    return contact.update({ firstName, lastName, phone })
      .then(() => res.send(contact))
      .catch((err) => {
        console.log('***Error updating contact', JSON.stringify(err))
        res.status(400).send(err)
      })
  })
});

app.listen(3000, () => {
  console.log('Server is up on port 3000');
});
````
````
npm start
````
