TASK DATE (MEDIUM): 14.03.2018 - FINISHED: 15.03.2018


TASK SHORT DESCRIPTION: 1417 [
								"Option to lock content (news articles and resources that are tagged to clubs and marked as 'private'):
								*Add a checkbox (""Hide resource from resources"") to resources that hides the resource from the resources page. 
									Add an info icon to this to explain ""Ticking this option will hide the resource from the public resources page. 
									This is useful if you wish to only show the resource to a tagged club.""
								*Add 'Private content' checkbox to the clubs edit form.
								*If a resource or news is tagged to the club and 'private content' is selected,
									A locked clubs page will show similar to how they are while logged out 
									(but with tagged resources and news articles hidden) - asking a user to join"
							]

							Summarized task

							1. I need to create a new option to the resources edit/create form: "club only resource"

							2. If this option is ticked: in that case resource can not be seen for everyone on the /resources page, but can be seen (for example) 
							   on /clubs/view/entrepreneurs for everyone.

							3. I need to create a new option to the clubs edit/create form: "private content" 

							4. If this option is ticked, in that case resources and news can be seen just for club members on /clubs/view/entrepreneurs as well.

							
GITHUB REPOSITORY CODE: feature/task-1417-options-to-lock-content-and-make-private


ORIGINAL WORK: https://github.com/BusinessBecause/network-site/tree/feature/task-1417-options-to-lock-content-and-make-private


CHANGES
 
	IN FILES:
		
		\network-site\addons\default\modules\clubs\controllers\clubs.php
		
			ADDED CODE: 
			
				Inside function view
				
					// get articles and resources about this club 
					#first checking club's private content option - if it's true, in that case club news can be seen for club members and administrators   
					$displayNewsAndResources = true;
					$clubResources = $clubArticles = array();
					if ( $club->private_content ) {
						$displayNewsAndResources = ($isMember or $is_admin) ? true : false; 
					}
					if ($displayNewsAndResources) {
						$this->load->library('news/news_lib');
						$clubArticles = $this->news_lib->get_posts_for_club($club->id);
						$clubResources = $this->resources_m->getClubResourcesList($club_slug);
					}
	
	
	
		\network-site\addons\default\modules\resources\models\resources_m.php
		
			CHANGED CODE: 
			
				Inside function get_resources_list
				
					FROM: $where = " WHERE `r`.`status`='live' ";
					TO: $where = " WHERE `r`.`status`='live'  AND `r`.`club_only_resource` = 0 ";
	
	
				Inside function get_most_recent_resources_list
				
					FROM: WHERE `r`.`status`='live'   
					TO: WHERE `r`.`status`='live'  AND `r`.`club_only_resource` = 0  
					
					
					
	
		\network-site\addons\default\modules\network_settings\controllers\content.php
			
			ADDED CODE: 
			
				inside function edit_club
				
					....
					$club['private_content'] = $this->input->post('private_content') ? 1 : 0;
					.....
	
	
	
		\network-site\addons\default\modules\network_settings\views\content\clubs_form.php
		
			ADDED CODE: 
			
				<!-- Private content - if it's ticked -> news and resources can be seen on public just for club members -->
				<li>
					<label for="private_content">
						Private content?
						<a class="my-tool-tip" data-toggle="tooltip" data-placement="right" title="" data-original-title="Ticking this option will make news and resources visible for club members only."> 
							<span class="glyphicon glyphicon-question-sign"></span>
						</a>
					</label>
					<div style="display: block; position: relative;">
						<div><?=form_checkbox('private_content', 1, isset($club) ? $club->private_content : 0, "id='private_content'")?></div>
					<div>
				</li>
								
	
	
	
		\network-site\addons\default\modules\network_settings\views\content_resources\resources_form.php
	
			ADDED CODE: 
			
				<!-- show or doesn't the resource on /resources page-->
				<li>
					<label for="club_only_resource">
						Club only resource?
						<a class="my-tool-tip" data-toggle="tooltip" data-placement="right" title="" data-original-title="Ticking this option will hide the resource from the public resources page."> 
							<span class="glyphicon glyphicon-question-sign" style="color: #999;"></span>
						</a>	
					</label>
					<div style="display: block; position: relative;">
						<div class="input">
							<?=form_checkbox('club_only_resource', 1, $post->club_only_resource)?>
						</div>
					<div>
				</li>
	

	
	
		\network-site\addons\default\modules\network_settings\controllers\content_resources.php
	
			CHANGED CODE FROM careers project: 
			
				FROM: ->set('careers_resources_section_title', $this->careers_setup_m->select_settings_value('careers_resources', 'section_title'))
				TO: ->set('careers_resources_section_title', $this->careers_setup_m->select_settings_value('careersResources', 'section_title'))
	
			ADDED CODE:
			
				inside function create_resource
				
					.... 
					'club_only_resource' => ( $this->input->post('club_only_resource') == "" ? $this->input->post('club_only_resource') : 0),
					.... 
					'club_only_resource' => (isset($_POST['club_only_resource']) ? $_POST['club_only_resource'] : 0),
					....
					
				inside edit_resource function 
				
					....
					//show or doesn't the resource on public /resources page
					$newData['club_only_resource'] =  isset($_POST['club_only_resource'])  ? $_POST['club_only_resource'] : 0;
					....
	
	
	
	
		\network-site\addons\default\modules\network_settings\details.php
		
			ADDED CODE: 
				
				inside upgrade function 
		
				if (version_compare($old_version, '2.0.85', 'lt')) {
					$this->db->add_boolean_field_to_table($this->db->dbprefix('resources'), $field = 'club_only_resource', $null = false, $default = false, $after = '');
					$this->db->add_boolean_field_to_table($this->db->dbprefix('clubs'), $field = 'private_content', $null = false, $default = false, $after = '');
				}

				inside function install 
				
				$this->db->add_boolean_field_to_table($this->db->dbprefix('resources'), $field = 'club_only_resource', $null = false, $default = false, $after = '');
				$this->db->add_boolean_field_to_table($this->db->dbprefix('clubs'), $field = 'private_content', $null = false, $default = false, $after = '');
				
				
		
		
		FROM CAREERS PROJECT: \network-site\addons\default\modules\careers\details.php

			private function _get_records_to_careers_setup_table() 
			{
				return	array (	
					array (		
						'section' => 'welcome',
						'settings' => array(
							'switch' => true,
							'section_title' => 'Use your Alumni network for careers support',	
							'background_image' => '',
							'background_image_positioning' => 'fit',
							'background_color' => '#ffffff',
							'background_color_transparency' => '100',
							'sub_heading' => 'Welcome to the Alumni Careers Page',
							'content_photo' => '',
							'introduction_text' => '',
							'help_box_text' => 'Send a request to the careers team',
							'help_box_switch' => true,
						), 	
					), 
					array (
						'section' => 'careersResources',
						'settings' => array(
							'switch' => true,
							'section_title' => 'Careers Resources',	
							'background_image' => '',
							'background_image_positioning' => 'fit',
							'background_color' => '#ffffff',
							'background_color_transparency' => '100',
						), 	
					), 
					array (
						'section' => 'careersGuides',
						'settings' => array(
							'switch' => true,
							'section_title' => 'Careers Guides',	
							'background_image' => '',
							'background_image_positioning' => 'fit',
							'background_color' => '#ffffff',
							'background_color_transparency' => '100',
						),
					), 
					array (
						'section' => 'mentoring',
						'settings' => array(
							'switch' => true,
							'section_title' => 'Mentoring',	
							'background_image' => '',
							'background_image_positioning' => 'fit',
							'background_color' => '#ffffff',
							'background_color_transparency' => '100',
							'sub_heading' => 'Welcome to the Mentoring Page',
							'introduction_text' => '',
						), 	
					), 
					array (
						'section' => 'featuredMentors',
						'settings' => array(
							'switch' => true,
							'section_title' => 'Featured Mentors',	
							'background_image' => '',
							'background_image_positioning' => 'fit',
							'background_color' => '#ffffff',
							'background_color_transparency' => '100',
						),
					), 
					array (
						'section' => 'jobsAndInternships',
						'settings' => array(
							'switch' => true,
							'section_title' => 'Jobs and Internships',	
							'background_image' => '',
							'background_image_positioning' => 'fit',
							'background_color' => '#ffffff',
							'background_color_transparency' => '100',
						),
					), 
					array (
						'section' => 'careersNews',
						'settings' => array(
							'switch' => true,
							'section_title' => 'Careers News',	
							'background_image' => '',
							'background_image_positioning' => 'fit',
							'background_color' => '#ffffff',
							'background_color_transparency' => '100',
						),
					), 
					array (
						'section' => 'careersGroups',
						'settings' => array(
							'switch' => true,
							'section_title' => 'Careers Groups',	
							'background_image' => '',
							'background_image_positioning' => 'fit',
							'background_color' => '#ffffff',
							'background_color_transparency' => '100',
						),
					), 
					array (
						'section' => 'careersEvents',
						'settings' => array(
							'switch' => true,
							'section_title' => 'Careers Events',	
							'background_image' => '',
							'background_image_positioning' => 'fit',
							'background_color' => '#ffffff',
							'background_color_transparency' => '100',
						),
					), 
				);
			}//END function _get_records_to_careers_setup_table	
