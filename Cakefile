fs = require 'fs-extra'
path = require 'path'

styleName = "Television"

getLocalPath = (pathComponents...) ->
  pathComponents.unshift(__dirname)
  path.join.apply(path, pathComponents)

bundleModel = ->
  model = { }
  model[prop] = "%#{prop}%" for prop in ["baseurl", "priority", "opacity", "title", "text", "image"]
  model.image = 'growlimage://' + model.image
  model[prop] = null for prop in ["bodyStyle"]
  model

loadModel = (file = "psudo") ->
  fullname = getLocalPath('source', "#{file}.json")
  model = fs.readJSONFileSync fullname
  model.baseurl = undefined # avoid to render base url
  model

compileJade = (sourceFile, targetDir, pretty, model) ->
  sourceCode = fs.readFileSync sourceFile, 'utf8'
  template = require('jade').compile(sourceCode, { pretty: pretty })
  html = template(model)
  targetFile = path.join(targetDir, path.basename(sourceFile, ".jade")) + ".html"
  fs.writeFileSync targetFile, html, 'utf8'

compileLess = (sourceFile, targetDir) ->
  targetFile = path.join(targetDir, path.basename(sourceFile, ".less")) + ".css"
  require('recess') sourceFile, { compile: true }, (err, result) ->
    fs.writeFileSync targetFile, result.output, 'utf8'

option '-p', '--psudo [file]', 'Psudo model file, default is psudo.json'
option '-w', '--watch', "Watch file change"

runWatch = (options, action) ->
  unless options.watch
    action(options)
    return
  console.info "INFO: Watching..."
  files = fs.readdirSync getLocalPath('source')
  console.log '"Tracking files:'
  for file in files
    console.log "  #{file}"
    fs.watchFile getLocalPath('source', file), (current, previous) ->
      unless current.mtime == previous.mtime
        console.log "#{file} Changed..."
        action(options)

task "bundle", "Build growlStyle Bundle", (options) ->
  rootPath = getLocalPath("#{styleName}.growlStyle")
  fs.mkdirsSync path.join(rootPath, "Contents", "Resources")
  fs.copy getLocalPath('source', 'info.plist'), path.join(rootPath, "Contents", "info.plist")
  compileJade getLocalPath('source', 'template.jade'), path.join(rootPath, "Contents", "Resources"), true, bundleModel()
  compileLess getLocalPath('source', 'default.less'), path.join(rootPath, "Contents", "Resources")

task "psudo", "Open fake page in browser", (options) ->
  runWatch options, ->
    fs.mkdirsSync getLocalPath('psudo')
    fs.copy getLocalPath('source','growl.png'), getLocalPath('psudo','growl.png') unless fs.existsSync getLocalPath('psudo','growl.png')
    compileJade getLocalPath('source', 'template.jade'), getLocalPath('psudo'), true, loadModel(options.psudo)
    compileLess getLocalPath('source', 'default.less'), getLocalPath('psudo')

task "live", "Deploy to installed package", (options) ->
  runWatch options, ->
    rootPath = path.join(process.env.HOME,'Library/Application Support/Growl/Plugins', "#{styleName}.growlStyle")
    compileJade getLocalPath('source', 'template.jade'), path.join(rootPath, "Contents", "Resources"), true, bundleModel()
    compileLess getLocalPath('source', 'default.less'), path.join(rootPath, "Contents", "Resources")

task "try", "Invoke notification", (options) ->
  runWatch options, ->
    require('growl') 'Preview Message',
      image: getLocalPath('source','growl.png')
      title: 'Preview Title'

