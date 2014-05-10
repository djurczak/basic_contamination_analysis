require 'json'

genome_index =
  "/groups/brennecke/annotation/genome/5_55/dmel-all-chromosome-r5.55.noUEx"
contamination_index =
  "/groups/brennecke/annotation/filters/annotation_old/indices/all_filters"

libs = "19092,19093,19094,19095,19096,19097,19098"

data = %x{
  curl -s http://piwi:3010/processed_locations?libs=#{libs}
}
parsed = JSON.parse(data)

INPUT_FILES = parsed.map { |p| p["location"] }
collapsed_files = INPUT_FILES.map do |f|
  File.join("tmp", File.basename(f.ext("collapsed")))
end

directory "tmp"
task :default => ["tmp", :collapsed]
task :collapsed => collapsed_files

rule '.unaligned' => ->(f) { input_source_for(f) } do |t|
  %x{
    . /etc/profile.d/modules.sh && module load bowtie &&
    bowtie -f -a -v3 -p18 --strata --best #{genome_index} #{t.source} \
-S --un tmp/#{File.basename(t.name)} > /dev/null
  }
end

rule '.noconta' => '.unaligned' do |t|
  %x{
    . /etc/profile.d/modules.sh && module load bowtie &&
    bowtie -f -a -v3 -p18 --strata --best #{contamination_index} #{t.source} \
-S --al #{t.name.ext('conta')} --un #{t.name} > /dev/null
  }
end

rule '.collapsed' => '.noconta' do |t|
  %x{
    . /etc/profile.d/modules.sh && module load fastx-toolkit &&
    fastx_collapser -i #{t.source} -o #{t.name}
  }
end

def input_source_for(file)
  INPUT_FILES.detect { |f| f.include?(File.basename(file.ext)) }
end
