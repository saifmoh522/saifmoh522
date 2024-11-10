const { Client, Collection } = require('discord.js-selfbot-v13');
const client = new Client({
  checkUpdate: false,
});

const fs = require('fs');
const prefix = '.';

const cloneFunctions = new Collection();

fs.readdirSync('./clonehandler').forEach((file) => {
  if (file.endsWith('.js')) {
    const functionName = file.replace('.js', '');
    const cloneFunction = require(`./clonehandler/${file}`);
    cloneFunctions.set(functionName, cloneFunction);
  }
});

client.on('ready', () => {
  console.log(`Cloner works on ${client.user.username}`);
  console.log('USE THIS AT YOUR OWN RISK');
});

const deleteEverything = require('./clonehandler/deleteeverything');
const cloneChannelsCategories = require('./clonehandler/channelscategories');
const rolesCreate = require('./clonehandler/rolescreate');
const channelsCategoriesPerms = require('./clonehandler/channelscategoriesperms');

client.on('messageCreate', async (message) => {
  if (!message.content.startsWith(prefix) || message.author.bot) return;

  const [command, serverId] = message.content.slice(prefix.length).trim().split(/ +/);

  if (command === 'clone') {
    if (!message.member.permissions.has('ADMINISTRATOR')) {
      return message.reply('معكش ادمنستريتور.');
    }

    if (!serverId) {
      return message.reply('وين ايدى السيرفر اللى بتنسخه.');
    }

    try {
      await deleteEverything(client, message.guild.id);
      console.log('Deleted everything in the clone guild.');
    } catch (error) {
      console.error('Failed to delete everything in the clone guild.');
      console.error(error);
    }

    try {
      await cloneFunctions.get('channelscategories')(serverId, message.guild.id, client);
      console.log('Executed clone function: channelscategories');
    } catch (error) {
      console.error('Failed to execute clone function: channelscategories');
      console.error(error);
    }

    try {
      await cloneFunctions.get('rolescreate')(serverId, message.guild.id, client);
      console.log('Executed clone function: rolescreate');
    } catch (error) {
      console.error('Failed to execute clone function: rolescreate');
      console.error(error);
    }

    try {
      await cloneFunctions.get('channelscategoriesperms')(serverId, message.guild.id, client);
      console.log('Executed clone function: channelscategoriesperms');

      await channelsCategoriesPerms(serverId, message.guild.id, client);
      console.log('Set permissions for channels and categories successfully!');
    } catch (error) {
      console.error('Failed to execute clone function: channelscategoriesperms');
      console.error(error);
    }
  }
});


client.login(process.env['TOKEN']);
//put your token in Tools -> Secrets-> TOKEN
//put your token in Tools -> Secrets-> TOKEN

module.exports = {
  client,
};
