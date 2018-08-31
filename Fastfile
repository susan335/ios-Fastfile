fastlane_version "2.97.0"

fastlane_require 'net/http'
fastlane_require 'openssl'
fastlane_require 'json'
fastlane_require 'fileutils'
fastlane_require 'mini_magick'

default_platform :ios

platform :ios do

  ####################################################
  #
  # For tests
  #
  ####################################################
  # ScanFile
  # - - - - - - - - - - - - - - -
  # scheme("SCHEME")
  # only_testing(["TARGET"])
  # - - - - - - - - - - - - - - -
  desc "Runs all the tests"
  lane :test do
    scan(
      device: "iPhone 8",
      clean: true,
      skip_slack: true
    )
  end

  # Snapfile
  # - - - - - - - - - - - - - - -
  # scheme("SCHEME")
  # xcargs "-only-testing:TARGET"
  # - - - - - - - - - - - - - - -
  desc "Runs snapshot tests"
  lane :screenshot_test do |options|
    mode = options[:mode] if options[:mode]
    mode = ENV["SCREENSHOT_TEST_MODE"] unless options[:mode]

    output_dir = "result"
    model_dir = "model"
    diff_dir = "diff"

    snapshot(
      devices: ["iPhone 8 Plus", "iPhone 8", "iPhone SE", "iPhone X"],
      languages: ["ja"],
      clean: false,
      output_directory: "./fastlane/snapshot_test/#{output_dir}"
    )
    hasDiff = false
    Dir.chdir("snapshot_test") do
      hasDiff = compare_image(
        dir1:"#{model_dir}/ja/",
        dir2:"#{output_dir}/ja/",
        output_dir:"#{diff_dir}/",
      )
      p hasDiff
      if mode == "verify"
        UI.user_error!("[ScreenShot] There are different pixcel.") if hasDiff == true
      elsif
        File.rename(model_dir, "#{model_dir}_old")
        File.rename(output_dir, model_dir)
      end
    end
  end

  desc "Compare images"
  private_lane :compare_image do |options|
    hasDiff = false
    FileUtils.mkdir_p("#{options[:output_dir]}", :mode => 0777)

    modelImages = Dir.glob("#{options[:dir1]}/*.png")
    compare_images = Dir.glob("#{options[:dir2]}/*.png")
    modelImages.each do |imageA|
      diffImage = "#{options[:output_dir]}/#{File.basename(imageA)}_diff.png"
      imageB = compare_images.find { |image| File.basename(imageA) == File.basename(image) }
      compare_images.delete(imageB)
      p imageA, imageB

      compare = MiniMagick::Tool::Compare.new(whiny: false)
      compare.metric('AE')
      compare << imageA
      compare << imageB
      compare << diffImage
      compare.call do |_, dist, _|
        p dist
        if dist == "0"
          File.delete(diffImage)
        end
        hasDiff = true unless dist.to_i.zero?
      end
    end
    hasDiff
  end

  ####################################################
  #
  # For archive
  #
  ####################################################
  # Matchfile
  # - - - - - - - - - - - - - - -
  # git_url("ssh://XXXX")
  # - - - - - - - - - - - - - - -
  # dotenv
  # - - - - - - - - - - - - - - -
  # MATCH_PASSWORD = "XXXXXXX"
  # - - - - - - - - - - - - - - -
  desc "Runs all the tests"
  private_lane :my_match do |options|
    UI.user error!("Required type.") unless options[:type]
    UI.user error!("Required bundle_id.") unless options[:bundle_id]
    match(
      force: true,
      clone_branch_directly: "master",
      type: "#{options[:type]}",
      app_identifier: options[:bundle_id]
    )
  end

  # Gymfile
  # - - - - - - - - - - - - - - -
  # workspace("XXXX.xcworkspace")
  # - - - - - - - - - - - - - - -
  desc "Builds and packages iOS apps"
  private_lane :my_gym do |options|
    UI.user error!("Required scheme name.") unless options[:scheme]
    UI.user error!("Required configuration.") unless options[:configuration]
    UI.user error!("Required provisioningProfiles.") unless options[:provisioningProfiles]
    UI.user error!("Required type.") unless options[:type]

    configuration = "#{options[:configuration]}"
    gym(
      scheme: "#{options[:scheme]}",
      configuration: configuration,
      clean: true,
      output_name: configuration + ".ipa",
      export_method: "#{options[:type]}",
      include_symbols: true,
      export_xcargs: "-allowProvisioningUpdates",
      export_options: {
        provisioningProfiles: options[:provisioningProfiles]
      }
    )
  end

  ####################################################
  #
  # For Deploy
  #
  ####################################################
  # dotenv
  # - - - - - - - - - - - - - - -
  # STAGE_SCHEME = "XXXXXXX"
  # STAGE_CONFIGURATION = "XXXXXXX"
  # STAGE_DISTRIBUTION_KEY = "XXXXXXX"
  # CRASHLYTICS_API_TOKEN = "XXXXXXX"
  # STAGE_BUNDLE_ID = "XXXXXXX"
  # - - - - - - - - - - - - - - -
  desc "Deploy a new Stage Build to DeployGate"
  lane :submit_stage_dg do
    submit_dg(
      scheme: ENV["STAGE_SCHEME"],
      configuration: ENV["STAGE_CONFIGURATION"],
      distribution_key: ENV["STAGE_DISTRIBUTION_KEY"],
      bundle_id: ENV["STAGE_BUNDLE_ID"]
    )
  end

  # dotenv
  # - - - - - - - - - - - - - - -
  # ADHOC_SCHEME = "XXXXXXX"
  # ADHOC_CONFIGURATION = "XXXXXXX"
  # ADHOC_DISTRIBUTION_KEY = "XXXXXXX"
  # CRASHLYTICS_API_TOKEN = "XXXXXXX"
  # ADHOC_BUNDLE_ID = "XXXXXXX"
  # - - - - - - - - - - - - - - -
  desc "Deploy a new Release Build to DeployGate"
  lane :submit_release_dg do
    submit_dg(
      scheme: ENV["ADHOC_SCHEME"],
      configuration: ENV["ADHOC_CONFIGURATION"],
      distribution_key: ENV["ADHOC_DISTRIBUTION_KEY"],
      bundle_id: ENV["ADHOC_BUNDLE_ID"]
    )
  end

  # dotenv
  # - - - - - - - - - - - - - - -
  # CRASHLYTICS_API_TOKEN = "XXXXXXX"
  # - - - - - - - - - - - - - - -
  desc "Deploy a new Build to DeployGate"
  private_lane :submit_dg do |options|
    UI.user error!("Required scheme name.") unless options[:scheme]
    UI.user error!("Required configuration.") unless options[:configuration]
    UI.user error!("Required distribution_key.") unless options[:distribution_key]
    UI.user error!("Required bundle id.") unless options[:bundle_id]
    my_match(
      type: "adhoc",
      bundle_id: "#{options[:bundle_id]}"
    )
    my_gym(
      scheme: "#{options[:scheme]}",
      configuration: "#{options[:configuration]}",
      type: "ad-hoc",
      provisioningProfiles: { options[:bundle_id] => "match AdHoc #{options[:bundle_id]}" }
    )
    deploygate(
      message: changelog_from_git_commits(pretty: "%h: %s", commits_count: 10),
      distribution_key: "#{options[:distribution_key]}"
    )
    upload_symbols_to_crashlytics(
      dsym_path: options[:configuration] + ".app.dSYM.zip",
      api_token: ENV["CRASHLYTICS_API_TOKEN"]
    )
  end

  # dotenv
  # - - - - - - - - - - - - - - -
  # RELEASE_SCHEME = "XXXXXXX"
  # RELEASE_CONFIGURATION = "XXXXXXX"
  # CRASHLYTICS_API_TOKEN = "XXXXXXX"
  # RELEASE_BUNDLE_ID = "XXXXXXX"
  # - - - - - - - - - - - - - - -
  desc "Deploy a new version to the App Store"
  lane :submit_store do
    my_match(
      type: "appstore",
      bundle_id: ENV["RELEASE_BUNDLE_ID"]
    )
    my_gym(
      scheme: ENV["RELEASE_SCHEME"],
      configuration: ENV["RELEASE_CONFIGURATION"],
      type: "app-store",
      provisioningProfiles: { ENV["RELEASE_BUNDLE_ID"] => "match #{ENV["RELEASE_BUNDLE_ID"]}" }
    )
    deliver(
      app_identifier: ENV["RELEASE_BUNDLE_ID"],
      ipa: "./#{ENV["RELEASE_CONFIGURATION"]}.ipa",
      force: true,
      automatic_release: false,
      phased_release: true,
      submit_for_review: true,
      submission_information: {
        "export_compliance_encryption_updated": false,
        "add_id_info_uses_idfa": false
      }
    )
  end

  # dotenv
  # - - - - - - - - - - - - - - -
  # INFO_PLIST_PATH = "XXXXXXX"
  # CRASHLYTICS_API_TOKEN = "XXXXXXX"
  # RELEASE_BUNDLE_ID = "XXXXXXX"
  # - - - - - - - - - - - - - - -
  desc "Download dSYM from iTunesConnect. and Upload to Crashlytics."
  lane :dsym do
    version_string = get_info_plist_value(path: ENV["INFO_PLIST_PATH"], key: "CFBundleShortVersionString")
    download_dsyms(
      version: version_string,
      app_identifier: ENV["RELEASE_BUNDLE_ID"]
    )
    upload_symbols_to_crashlytics(
      api_token: ENV["CRASHLYTICS_API_TOKEN"]
    )
  end

  desc "Register new device. ex) fastlane regist_new_device name:XXX udid:YYY"
  lane :regist_new_device do |options|
    register_device(options)
  end

  ####################################################
  #
  # For Git supports
  #
  ####################################################
  # dotenv
  # - - - - - - - - - - - - - - -
  # INFO_PLIST_PATH = "XXXXXXX"
  # - - - - - - - - - - - - - - -
  desc "Create new release branch,  ex: fastlane release_branch version:1.0.0"
  lane :git_release_branch do |options|
    UI.user error!("Required release version. ex: fastlane release_branch version:1.0.0") unless options[:version]
    version_string = "#{options[:version]}"
    branch = "release/#{version_string}"

    # リリースバージョンが書かれたpropertiesファイルを更新
    set_info_plist_value(path: ENV["INFO_PLIST_PATH"], key: "CFBundleShortVersionString", value: version_string)
    # リリースブランチを作成
    sh("git checkout -b #{branch}")
    # commitを作成
    sh("git add ../#{ENV["INFO_PLIST_PATH"]}")
    sh("git commit -m \"Update version string to #{version_string}\"")
    sh("git push --set-upstream origin #{branch}")
  end

  # dotenv
  # - - - - - - - - - - - - - - -
  # INFO_PLIST_PATH = "XXXXXXX"
  # - - - - - - - - - - - - - - -
  desc "Create new tag"
  lane :git_release_tag do
    # propertiesファイルからバージョン名を取得
    version_string = get_info_plist_value(path: ENV["INFO_PLIST_PATH"], key: "CFBundleShortVersionString")
    puts "version string = #{version_string}"
    # tagつけてpush!
    sh("git tag #{version_string}")
    sh("git push origin #{version_string}")
  end

  ####################################################
  #
  # For GitLab Trigger
  #
  ####################################################
  # dotenv
  # - - - - - - - - - - - - - - -
  # GITLAB_HOST = "gitlab.com"
  # GITLAB_PROJ_ID = "XXX"
  # CI_TOKEN = "XXXXXXXX"
  # - - - - - - - - - - - - - - -
  desc "Trigger ci job."
  private_lane :post_ci_trigger do |options|
    variable = options[:variable]
    host = ENV["GITLAB_HOST"]
    url = URI.parse("https://#{host}/api/v4/projects/#{ENV["GITLAB_PROJ_ID"]}/trigger/pipeline")
    request = Net::HTTP::Post.new(url)

    request.set_form_data({ "token" => "#{ENV["CI_TOKEN"]}", "ref" => git_branch(), "variables[#{variable}]" => true })
    https = Net::HTTP.new(host, 443)
    https.use_ssl = true
    response = https.start {|http| http.request(request) }
    puts response.body
  end

  desc "Trigger ci job for accept screenshot diff."
  lane :trigger_update_screenshot do
    post_ci_trigger variable:"ACCEPT_SCREENSHOT_DIFF"
  end

  desc "Trigger ci job for deploy staging app."
  lane :trigger_submit_stage_dg do
    post_ci_trigger variable:"RUN_SUBMIT_STAGE_TO_DG"
  end

  desc "Trigger ci job for deploy release app."
  lane :trigger_submit_release_dg do
    post_ci_trigger variable:"RUN_SUBMIT_RELEASE_TO_DG"
  end

  desc "Trigger ci job for submit appstore."
  lane :trigger_submit_appstore do
    post_ci_trigger variable:"RUN_SUBMIT_RELEASE_TO_APPSTORE"
  end

  desc "Run 'pod update' at a ci environment."
  lane :trigger_pod_update do
    post_ci_trigger variable:"RUN_POD_UPDATE"
  end
end
