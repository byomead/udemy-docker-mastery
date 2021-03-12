# create your custom drupal image here, based of official drupal
# Base image
FROM drupal:8

# Install Git
RUN apt-get update && apt-get install -y git \
  && rm -rf /var/lib/apt/lists/*

# Move to another dir
WORKDIR /var/www/html/themes

# Clone the drupal theme and set the default drupal user as owner of the filess
RUN git clone --branch 8.x-3.x --single-branch --depth 1 https://git.drupalcode.org/project/bootstrap.git \
  && chown -R www-data:www-data bootstrap

# Change workdir to default
WORKDIR /var/www/html