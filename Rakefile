require 'docker'

DOCKER_REGISTRY_USER = ENV['DOCKER_REGISTRY_USER']
DOCKER_REGISTRY_PASS = ENV['DOCKER_REGISTRY_PASS']
DOCKER_REGISTRY_EMAIL = ENV['DOCKER_REGISTRY_EMAIL']

IMAGE_NAME = 'blackpuppy-apiproxy'

task :build_image do
	p "Building Docker Image:- blackpuppy-apiproxy"

	Docker.options = {:read_timeout => 3600}
	image = Docker::Image.build_from_dir './', {:t => "#{IMAGE_NAME}"}
end

task :push do
	p "Push image to registry"

	image =  find_image "#{IMAGE_NAME}:latest"
	image.tag('repo' => "tutum.co/nicholasjackson/#{IMAGE_NAME}", 'force' => true) unless image.info["RepoTags"].include? "tutum.co/nicholasjackson/#{IMAGE_NAME}:latest"

  sh "docker login -u #{DOCKER_REGISTRY_USER} -p #{DOCKER_REGISTRY_PASS} -e #{DOCKER_REGISTRY_EMAIL} https://tutum.co"
  sh "docker push tutum.co/nicholasjackson/#{IMAGE_NAME}:latest"
end

def get_docker_ip_address
	if !ENV['DOCKER_HOST']
		return "127.0.0.1"
	else
		# dockerhost set
		host = ENV['DOCKER_HOST'].dup
		host.gsub!(/tcp:\/\//, '')
		host.gsub!(/:2376/,'')

		return host
	end
end

def find_image image_name
	found = nil
	Docker::Image.all.each do | image |
		found = image if image.info["RepoTags"].include? image_name
	end

	return found
end

def get_container
	container = find_running_container
	if container != nil
		p "found #{container}"
		return container
	else
		return create_and_start_container
	end
end

def find_running_container
	containers = Docker::Container.all(:all => true)
	found = nil

	containers.each do | container |
		if container.info["Image"] == "gobuildserver"
			found = container
		end
	end

	return found
end

def create_and_start_container
	# update the timeout for the Excon Http Client
	# set the chunk size to enable streaming of log files
	Docker.options = {:chunk_size => 1, :read_timeout => 3600}

	command = ['/bin/bash']
	container = Docker::Container.create(
		'Image' => 'gobuildserver',
		'Cmd' => command,
		'Tty' => true,
		"Binds" => [
			"#{ROOTFOLDER}:/src",
			"#{ROOTFOLDER}/go/src:/go/src"
		],
		"Env" => [
			"API_CONFIG=/src/devconfig.json",
			"API_ROOT=/src"
		],
		'WorkingDir' => '/go/src/github.com/transform/transformapi')

	container.start

	return container
end
